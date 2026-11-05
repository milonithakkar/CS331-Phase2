# Reading guide: TLS, fingerprints and QUIC

What we read before writing the extractor and the tests, and what to take from each part. For exact details, use the RFC sections themselves; they are also what the report cites.

Order: Part I before touching the QUIC extractor, Part II before the fingerprint tests, Part III only if we add QUIC v2.

## Part I: what the QUIC extractor needs

### 1. Handshake messages, TLS records and CRYPTO frames

A TLS handshake message starts with a 1-byte type and a 3-byte length. The ClientHello is one such message. Over TCP, handshake messages are carried in TLS records, which are carried in TCP segments, and the boundaries of these do not have to line up.

QUIC uses the TLS 1.3 handshake but not TLS records. The handshake bytes go into CRYPTO frames, and each CRYPTO frame says at which offset its bytes belong. So the extractor has to:

1. find a QUIC Initial packet and its length,
2. decrypt it,
3. find the CRYPTO frames in it,
4. put each piece at its offset, not in arrival order,
5. wait until the 4-byte header and the full declared length are there,
6. parse the ClientHello.

Keep connections and directions apart; never mix pieces from different ones in one buffer. Searching a UDP payload for bytes that look like a ClientHello does not work, because the payload is encrypted and the hello can be split.

Read: [RFC 8446 §4.1.2](https://www.rfc-editor.org/rfc/rfc8446.html#section-4.1.2) (ClientHello), [RFC 9001 §4.1](https://www.rfc-editor.org/rfc/rfc9001.html#section-4.1) (TLS over QUIC), [RFC 9000 §19.6](https://www.rfc-editor.org/rfc/rfc9000.html#section-19.6) (CRYPTO frames).

### 2. QUIC packets

One UDP datagram can hold more than one QUIC packet. A long-header packet has a version, destination and source connection IDs, and for Initial packets a token. Its length field says where the packet ends, and that has to be respected before decrypting.

QUIC integers are variable-length: the first two bits give the size. The packet number is sent shortened and protected; after removing header protection you rebuild the full number from the largest one seen so far.

Read: [RFC 9000 §12.2](https://www.rfc-editor.org/rfc/rfc9000.html#section-12.2) (coalesced packets), [§16](https://www.rfc-editor.org/rfc/rfc9000.html#section-16) (integers), [§17.1](https://www.rfc-editor.org/rfc/rfc9000.html#section-17.1) (packet numbers), [§17.2.2](https://www.rfc-editor.org/rfc/rfc9000.html#section-17.2.2) (Initial packets).

### 3. Initial keys, HKDF, AEAD, header protection

HKDF has two steps. Extract turns input key material and a salt into a pseudorandom key. Expand turns that key plus a label into as many bytes as you need. TLS 1.3 adds a labelled version, HKDF-Expand-Label, and QUIC uses it.

For QUIC v1 Initial packets, the public salt and the client's first destination connection ID give an initial secret. From it come a client secret and a server secret, and from each a key, an IV and a header protection key. Anyone watching can do this, so Initial encryption protects against tampering but not against reading. Later QUIC packets use real secrets that an observer does not have.

The payload is AES-128-GCM: the header up to the packet number is checked but not encrypted, the payload is encrypted and checked. If the tag check fails, the packet is dropped. The nonce comes from the IV and the packet number, so a wrong packet number means a failed tag.

Header protection is separate. A sample of the encrypted payload gives a mask that hides some bits of the first byte and the packet number. You remove that mask first, then decrypt.

Read: [RFC 5869 §2](https://www.rfc-editor.org/rfc/rfc5869.html#section-2.1) (HKDF), [RFC 8446 §7.1](https://www.rfc-editor.org/rfc/rfc8446.html#section-7.1) (HKDF-Expand-Label), [RFC 9001 §5.2](https://www.rfc-editor.org/rfc/rfc9001.html#section-5.2) (Initial secrets), [§5.3 to §5.4](https://www.rfc-editor.org/rfc/rfc9001.html#section-5.3) (packet and header protection), [RFC 5116 §2](https://www.rfc-editor.org/rfc/rfc5116.html#section-2) (AEAD).

We use OpenSSL 3 (EVP for AES-GCM, the KDF interface for HKDF) instead of writing any crypto ourselves, and test against the RFC 9001 Appendix A values, because an API call succeeding does not mean the inputs were right.

### 4. Reassembly and safe parsing

CRYPTO frames are byte ranges with offsets. The reassembler should accept pieces in any order, accept an identical repeated piece, reject a piece that conflicts with bytes already stored, report gaps, cap the amount of memory, and check `offset + length` for overflow. Every length field in a packet is untrusted, so check it before reading.

In the C++ code we use `std::span<const std::uint8_t>`, fixed-width integers and explicit bounds checks, and return an error rather than a half-parsed hello.

## Part II: fingerprints and what an observer can see

### 5. JA3 and JA4

A TLS fingerprint summarises some fields of the handshake. It points to a TLS library and its settings, not to a unique app or person. Apps can share a library, and one app can change its hello between versions or connections.

JA3 joins the version, ciphers, extensions, groups and point formats in the order sent, drops GREASE, and takes the MD5. JA3S does the same for the ServerHello. JA4 has three parts: a readable prefix, a hash of the sorted ciphers and a hash of the sorted extensions and signature algorithms. The sorting and GREASE rules are part of the JA4 spec and have to be read, not guessed from output. JA4T reads the TCP SYN, JA4X the certificate and JA4H the HTTP request, so they need different things to be visible.

Read: [Salesforce JA3](https://github.com/salesforce/ja3), [FoxIO JA4](https://github.com/FoxIO-LLC/ja4), the [TShark manual](https://www.wireshark.org/docs/man-pages/tshark.html). Always note which tool and version produced a fingerprint. Our field-by-field notes are in [`landscape.md`](landscape.md).

### 6. GREASE

GREASE values are reserved values that clients send so servers learn to ignore unknown ones. A fingerprint has to recognise them and follow its own rule for each list. Tools do not always apply the same rule to every list; that is the cause of the 15 vs 1 JA4 result in T5.

Read: [RFC 8701](https://www.rfc-editor.org/rfc/rfc8701.html).

### 7. TCP fingerprints and capture point

A TCP fingerprint reads the SYN and its options (RFC 9293). If the capture starts after the handshake, there is no SYN to read. A proxy that opens a new TCP connection shows its own TCP stack, not the client's. Offload, NAT and the capture location all change what is seen, so record them. A difference caused by how we captured must not be reported as a change in the client.

Read: [RFC 9293](https://www.rfc-editor.org/rfc/rfc9293.html).

### 8. IPv6

IPv6 changes the IP header, the MSS and how addresses are assigned, and privacy addresses change over time. None of that changes the ClientHello bytes, so a fingerprint made only from the ClientHello should be the same over IPv4 and IPv6. Keep three questions apart: did the TLS fields change, did the TCP/IP fields change, and did tracking a device get harder?

Read: [RFC 8200](https://www.rfc-editor.org/rfc/rfc8200.html), [RFC 8201](https://www.rfc-editor.org/rfc/rfc8201.html), [RFC 8981](https://www.rfc-editor.org/rfc/rfc8981.html).

### 9. Certificate compression and abridged certificates

RFC 8879 compresses the certificate message. Abridged certificates are a newer proposal that sends a reference instead of the certificate. Both matter only if you can see the certificate, which in TLS 1.3 you cannot without keys. Since abridged certificates are not deployed, we only discuss them, we do not claim a measured result.

Read: [RFC 8879](https://www.rfc-editor.org/rfc/rfc8879.html).

### 10. Encrypted DNS

DoH and DoQ encrypt DNS lookups. That does not change the ClientHello of the next connection, but it stops an observer from linking a plaintext DNS lookup to that connection. It changes what can be linked, not the fingerprint.

Read: [RFC 8484](https://www.rfc-editor.org/rfc/rfc8484.html) (DoH), [RFC 9250](https://www.rfc-editor.org/rfc/rfc9250.html) (DoQ).

### 11. ECH

ECH encrypts the real ClientHello (with HPKE) inside an outer ClientHello that is still sent in the clear. An observer sees the outer hello's fields but not the real server name. Check which fields are in the outer hello rather than assuming everything disappears.

Read: [RFC 9849](https://www.rfc-editor.org/rfc/rfc9849.html) (ECH), [RFC 9180](https://www.rfc-editor.org/rfc/rfc9180.html) (HPKE).

### 12. HTTP fingerprints

JA4H works above TLS. HTTP/2 compresses headers with HPACK and HTTP/3 with QPACK. Without the TLS keys an observer cannot read the request at all, which is why JA4H needs keys and JA4 does not.

Read: [RFC 9113](https://www.rfc-editor.org/rfc/rfc9113.html) (HTTP/2), [RFC 9204](https://www.rfc-editor.org/rfc/rfc9204.html) (QPACK).

## Part III: QUIC v2 (optional)

QUIC v2 has its own constants and packet type numbers, so v1 values cannot be reused. Check the version first and keep separate test vectors. We did not implement it.

Read: [RFC 9369](https://www.rfc-editor.org/rfc/rfc9369.html).

## Questions we used to check ourselves

1. Draw the path from a UDP datagram to a parsed ClientHello, with every length and offset field.
2. Why can anyone decrypt QUIC Initial packets, and why does that not apply to later packets?
3. Two CRYPTO pieces arrive in reverse order. Where do their bytes go? What about an identical overlap, a conflicting overlap and a gap?
4. Pick one JA3 or JA4 field: which bytes, which normalisation rule, which spec, which RFC section?
5. For JA4, JA4T, JA4X and JA4H: which packet, which capture condition, and are keys needed?
6. Why can DoH hide a site name without changing the fingerprint?
7. What is the difference between certificate compression and abridged certificates?
8. Why is the same TLS fingerprint not proof of the same app or user?
