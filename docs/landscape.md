# Fingerprint landscape: JA3, JA3S, JA4, JA4S, JA4T, JA4X, JA4H

Before testing where the tools fail, we listed what each fingerprint reads, how it turns that into a string, which RFC defines the fields, and what a passive observer can see without TLS keys. One card per fingerprint is in [`basics/`](basics/README.md).

None of these fingerprints is an IETF standard. JA3 and JA3S come from Salesforce ([repository](https://github.com/salesforce/ja3)); the JA4+ family comes from FoxIO ([v1.0.1 release](https://github.com/FoxIO-LLC/ja4/releases/tag/v1.0.1), [technical details](https://github.com/FoxIO-LLC/ja4/blob/v1.0.1/technical_details/README.md)). The RFCs define the protocol fields, and the fingerprint authors decide which fields to use, how to sort them, what to do with GREASE, and how to hash. That choice can change between tool versions, which is exactly what we ran into in T5.

## Comparison

| | Reads | Fields | How it is built | Needs TLS keys? |
|---|---|---|---|---|
| JA3 | ClientHello | version, ciphers, extensions, groups, point formats | comma-joined in the order sent, GREASE removed, MD5 | no |
| JA3S | ServerHello | version, chosen cipher, extensions | comma-joined in order, MD5 | no |
| JA4 | ClientHello (TCP or QUIC) | transport, version, SNI present, cipher and extension counts, ALPN, ciphers, extensions, signature algorithms | `a_b_c`: readable prefix, hash of sorted ciphers, hash of sorted extensions + signature algorithms | no |
| JA4S | ServerHello (TCP or QUIC) | transport, version, extension count, ALPN, chosen cipher, extensions | readable prefix, cipher, hash of extensions in order | no |
| JA4T | TCP SYN | window, TCP option kinds, MSS, window scale | plain text, not hashed, option order kept | no, but depends on capture point |
| JA4X | each X.509 certificate | issuer and subject fields, extension OIDs | three truncated SHA-256 hashes | TLS 1.2 no, TLS 1.3 yes |
| JA4H | HTTP request | method, version, headers, cookies, language | readable prefix and hashes | yes for HTTPS |

What each one describes:

- JA3 and JA4 describe the TLS library and its settings on the client, not the person or the exact app. Apps that share a library can share a fingerprint (curl and Python share JA4 part b in T5), and one app can give many (Chromium gives 50 JA3s in T5).
- JA3S and JA4S depend on what the client offered as well as on the server.
- JA4T describes the TCP stack nearest to the capture point. A NAT keeps it; a proxy that ends the TCP connection replaces it (T3).
- JA4X describes how a certificate was built. JA4H describes one HTTP request.

## Which RFC defines what

| RFC | What it defines that the fingerprints use |
|---|---|
| [RFC 5246 §7.4.1](https://www.rfc-editor.org/rfc/rfc5246.html#section-7.4.1.2), [RFC 8446 §4.1](https://www.rfc-editor.org/rfc/rfc8446.html#section-4.1.2) | ClientHello and ServerHello (TLS 1.2, TLS 1.3) |
| [RFC 6066 §3](https://www.rfc-editor.org/rfc/rfc6066.html#section-3) | SNI |
| [RFC 7301](https://www.rfc-editor.org/rfc/rfc7301.html) | ALPN |
| [RFC 8422 §5.1](https://www.rfc-editor.org/rfc/rfc8422.html#section-5.1) | supported groups and EC point formats |
| [RFC 8446 §4.2.3](https://www.rfc-editor.org/rfc/rfc8446.html#section-4.2.3) | signature algorithms |
| [RFC 8701](https://www.rfc-editor.org/rfc/rfc8701.html) | GREASE values (the RFC defines them; each fingerprint decides whether to drop them) |
| [RFC 9849](https://www.rfc-editor.org/rfc/rfc9849.html), [RFC 9460](https://www.rfc-editor.org/rfc/rfc9460.html) | ECH, and the DNS HTTPS record that carries its key |
| [RFC 9000 §7.2](https://www.rfc-editor.org/rfc/rfc9000.html#section-7.2), [RFC 9001 §5.2](https://www.rfc-editor.org/rfc/rfc9001.html#section-5.2) | QUIC connection IDs and the public Initial keys |
| [RFC 9293 §3.7.1](https://www.rfc-editor.org/rfc/rfc9293.html#section-3.7.1), [RFC 6691](https://www.rfc-editor.org/rfc/rfc6691.html) | TCP options and MSS |
| [RFC 5280](https://www.rfc-editor.org/rfc/rfc5280.html), [RFC 8446 §4.4.2](https://www.rfc-editor.org/rfc/rfc8446.html#section-4.4.2) | X.509 certificates and the TLS Certificate message |
| [RFC 9110](https://www.rfc-editor.org/rfc/rfc9110.html), [RFC 9113](https://www.rfc-editor.org/rfc/rfc9113.html), [RFC 9114](https://www.rfc-editor.org/rfc/rfc9114.html), [RFC 6265](https://www.rfc-editor.org/rfc/rfc6265.html) | HTTP, HTTP/2, HTTP/3, cookies |
| [RFC 8484](https://www.rfc-editor.org/rfc/rfc8484.html) | DNS over HTTPS |

## What a passive observer can see

| What | Without keys | With keys |
|---|---|---|
| TCP SYN (JA4T) | yes, but it depends on where you capture and on offload | keys do not matter |
| TCP ClientHello (JA3, JA4) | yes; with ECH only the outer ClientHello | the ECH inner hello needs the server's ECH key, not the session keys |
| QUIC v1 ClientHello | yes; the Initial keys come from the connection ID in the packet (RFC 9001 §5.2) | not needed |
| ServerHello (JA3S, JA4S) | yes | not needed |
| TLS 1.3 certificate (JA4X) | no | yes |
| HTTPS request (JA4H) | no | yes |
| DNS name | yes over plain DNS, no over DoH | - |

Seeing one layer does not mean seeing the next. Decrypting QUIC Initial packets gives the ClientHello, but not the HTTP/3 request that follows.

## What our tests found, per fingerprint

| Fingerprint | Test | Finding |
|---|---|---|
| JA3 | T5 | Chromium: 50 JA3s in 50 connections, because of extension shuffling |
| JA4 | T1 | TShark and FoxIO both read the QUIC ClientHello, 5/5 per client |
| JA4 | T4 | a post-quantum key share changes JA4 through side effects (12 vs 14 extensions) |
| JA4 | T5 | the same 50 Chromium hellos give 15 JA4s with TShark 4.6.8 or FoxIO v0.18.6 and 1 with FoxIO v1.0.1, because of GREASE in the signature algorithms |
| JA4 | T7 | ECH adds one extension, so JA4 changes |
| JA4T / JA4TS | T3 | inside Docker the SYN-ACK values come from Docker's network stack; on the Wi-Fi card the container's SYN has Windows values |
| JA4X, JA4H | T2 | 0 without keys, 5 with keys on TLS 1.3 |

## Rules we followed when reading results

1. Say where the capture was taken. A fingerprint describes what was seen at that point.
2. Record the tool and its version. FoxIO's Rust ja4 reads packets through TShark, so the two agreeing is not two independent parsers.
3. JA3 keeps the order of lists; JA4 sorts them. JA4S and JA4T do not sort. GREASE handling depends on the fingerprint and on the tool version.
4. A hash is a short label, not a unique ID. Different clients can share one, and one client can change it after an update.
5. Do not decide there is a proxy from one MSS or TTL value. In T3 we only said so after seeing the same connection at two capture points.

## Other references

- [FoxIO JA4 Python source](https://github.com/FoxIO-LLC/ja4/blob/v1.0.1/python/ja4.py), [JA4T Rust source](https://github.com/FoxIO-LLC/ja4/blob/v1.0.1/rust/ja4/src/tcp.rs), [JA4H](https://github.com/FoxIO-LLC/ja4/blob/v1.0.1/python/ja4h.py), [JA4X](https://github.com/FoxIO-LLC/ja4/blob/v1.0.1/python/ja4x.py)
- [p0f](https://github.com/p0f/p0f), an older passive TCP/IP OS fingerprinting tool, for comparison with JA4T
