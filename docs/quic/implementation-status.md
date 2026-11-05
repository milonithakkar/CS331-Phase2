# C++ QUIC Initial extractor: what it does and how it was checked

The extractor reads a pcap, finds the client's QUIC v1 Initial packets, decrypts them, puts the CRYPTO frames back together and prints the complete ClientHello bytes. It does not compute JA4; that is left to FoxIO's tool.

## What it supports

- Classic pcap (v2.x) with Ethernet frames, IPv4 and IPv6 without extension headers, UDP.
- QUIC v1 long-header packets.
- Initial key derivation from the connection ID (RFC 9001 §5.2), header protection removal (§5.4) and AES-128-GCM decryption, through OpenSSL 3.
- CRYPTO frames placed by offset, so out-of-order pieces work. Identical overlaps are accepted, conflicting ones rejected, and there is a size limit.
- A ClientHello is printed only when all bytes from offset 0 up to its declared length are there, and its structure checks out.

The key derivation is tested against the client Initial values in RFC 9001 Appendix A.1.

Not supported: pcapng, VLAN tags, IP fragments, IPv6 extension headers, QUIC v2, Retry, connection migration, 0-RTT, and JA4.

## How it works

1. Parse QUIC variable-length integers and long headers, with bounds checks.
2. Derive the Initial keys. In the default (stateless) mode they come from each packet's own destination connection ID. In `--stateful-initial-context` mode they come from the first client Initial's connection ID on that UDP 4-tuple.
3. Remove header protection, recover the packet number, then decrypt and check the tag.
4. Parse the frames and collect CRYPTO ranges. Unknown frame types are an error.
5. Reassemble by offset and check the ClientHello.
6. Print counts per connection: Initial packets, decrypted, failed, and for incomplete ClientHellos the expected length and the missing byte ranges.

## The connection ID switch

A header-only inventory of the two QUIC pcaps (`code/scripts/quic_initial_inventory.py`, output in `results/track2/quic-initial-dcid-inventory.json`) shows five UDP flows per capture. In each flow the client's first Initial uses an 8-byte connection ID. After the server replies, the client's later Initials use a 20-byte ID that equals the server's source connection ID. For example, Chromium flow `172.17.0.2:36822` starts with `97726dafe5a69bee` and later uses `01c803772feafd9e11ca9d7770eddf9e11b1e424`.

RFC 9001 §5.2 says the Initial keys come from the destination connection ID of the client's first Initial. The server's ID does not change that. So the stateless mode, which derives keys from each packet's own ID, fails to decrypt every one of those later packets. That is where the extra failing connection IDs in its output came from.

The stateful mode (`code/quic/src/initial_context.cpp`) keeps, for each UDP 4-tuple, the first client connection ID, the client's source ID, and the server's source IDs it learns from server Initials. A later client Initial is decrypted with the first ID's keys only if its destination ID is the first ID or one of the learned server IDs. Unknown IDs never replace the stored one. Client and server packet numbers are tracked separately.

## Results

From the WSL run in `results/track2/verification-batch-20261105-105416/` (TShark 4.6.4, GCC 15.2.0, OpenSSL 3.5.5):

| capture | client Initials | stateless: decrypted / failed | stateful: decrypted / failed | server Initials decrypted | ClientHellos equal to TShark |
|---|---|---|---|---|---|
| curl | 20 | 10 / 10 | 20 / 0 | 5 / 5 | 5 / 5 |
| Chromium | 16 | 10 / 6 | 16 / 0 | 5 / 5 | 5 / 5 |

The comparison is on the exact bytes, as a set, against TShark's ClientHellos from the same pcap. TShark already decrypts these packets, so the stateful mode makes our extractor match TShark. In these captures the later packets only carry ACK and PADDING, so both modes already found all 5 ClientHellos; the extra decrypted packets do not change the ClientHello or the JA4.

### Damaged copies

`code/scripts/quic_impairment.py` makes damaged copies of each capture with a fixed seed (331) and compares the extractor with TShark on the same damaged file:

| damage | stateless | stateful |
|---|---|---|
| none | 5 / 5 | 5 / 5 |
| reversed packet order | 5 / 5 | 0 / 5 |
| shuffled order | 5 / 5 | 4 / 5 |
| one CRYPTO packet dropped | 5 / 5 | 5 / 5 |
| every third CRYPTO packet dropped | 2 / 2 | 2 / 2 |
| one CRYPTO packet cut by 64 bytes | 5 / 5 | 5 / 5 |

The numbers are the same for curl and Chromium. In total the stateless mode matched 54 of 54 and the stateful mode 42 of 54.

- When the order is reversed or shuffled, a later client packet can arrive first. The stateful mode then takes its connection ID as the key source and rejects the real first packet. This is a real weakness of the stateful mode.
- When every third packet is dropped, TShark itself only finds 2 complete ClientHellos per capture, so 2/2 means we agree with TShark, not that we recovered all 5.
- The single-packet drop and the 64-byte cut both hit the same packet, and TShark still finds all 5 ClientHellos afterwards, so that packet carried no ClientHello bytes.

## What could be done next

1. In stateful mode, fall back to the packet's own connection ID when it is unknown, so reordering does not lose ClientHellos.
2. Handle QUIC Retry: check the Retry integrity tag and switch the Initial keys to the new connection ID (RFC 9001 §5.2, §5.8).
3. Follow a connection across a change of UDP 4-tuple (migration, NAT rebinding), with tests in both directions.

References: [RFC 9001 §5.2](https://www.rfc-editor.org/rfc/rfc9001.html#section-5.2), [RFC 9000 §7.2](https://www.rfc-editor.org/rfc/rfc9000.html#section-7.2).
