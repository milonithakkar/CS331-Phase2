# Limitation ledger

One line per limitation. The commands, details and what each test does not show are in the linked test file.

Track 1 numbers come from the committed pcaps (`results/summary.md`, re-checked in [`results/track1/recheck-20261106.md`](results/track1/recheck-20261106.md)). Track 2 numbers come from the WSL run in `results/track2/verification-batch-20261105-105416/`.

| ID | Question | Test | Main number | Status |
|---|---|---|---|---|
| L1 | Can real tools read the QUIC ClientHello? | [T1](docs/tests/T1_quic.md) | TShark and FoxIO: 5/5 hellos for curl and for Chromium (Chromium's hello is in 21 CRYPTO pieces) | The tools handle it; our hypothesis was wrong |
| L2 | Which fingerprints need TLS keys? | [T2](docs/tests/T2_keys.md) | JA4X + JA4H: 0 without keys, 5 with keys (one TLS 1.3 connection) | Confirmed |
| L3 | Does the capture point change what we see? | [T3](docs/tests/T3_vantage_offload.md) | Offload on: hello in 1 frame of 1586 B; off: 1514 + 138 B. The SYN-ACK seen inside Docker (MSS 65495) comes from Docker's network stack. On Windows the container's SYN reaches the Wi-Fi card with Windows values (TTL 128, window scale 8) | Confirmed in our two labs |
| L4 | Does a post-quantum key share change the hello? | [T4](docs/tests/T4_post_quantum.md) | Hello 508 B (X25519) vs 1511 B (X25519MLKEM768); JA4 has 14 vs 12 extensions | Confirmed, through side effects |
| L5 | Does one browser give one fingerprint? | [T5](docs/tests/T5_stability.md) | Chromium, 50 hellos: 50 JA3s; JA4: 15 with TShark 4.6.8 or FoxIO v0.18.6, 1 with FoxIO v1.0.1 | Confirmed: depends on the tool version |
| L6 | IPv4 vs IPv6 | [T6](docs/tests/T6_ipv6.md) | IPv6 pcap has 0 packets (no IPv6 route on campus or on WSL) | Not measured |
| L7 | Does ECH hide the site? | [T7](docs/tests/T7_ech_dns.md) | ECH on: SNI `cloudflare-ech.com`, but plain DNS still asks for `crypto.cloudflare.com`. The DoH run did not load | Confirmed; DoH not measured |

Certificate compression and abridged certificates were only read about, not measured.

## Track 2: our C++ QUIC Initial extractor

Same two QUIC pcaps, compared with TShark 4.6.4 on the same file.

| | curl | Chromium |
|---|---|---|
| client Initial packets | 20 | 16 |
| decrypted, keys from each packet's own connection ID (stateless) | 10 | 10 |
| decrypted, keys from the connection's first connection ID (stateful) | 20 | 16 |
| complete ClientHellos, both modes | 5/5, same bytes as TShark | 5/5, same bytes as TShark |

- The packets that fail in stateless mode are the client packets sent after the server picked its own connection ID (RFC 9000 §7.2). RFC 9001 §5.2 says their keys still come from the first connection ID.
- In these pcaps those packets carry only ACK and PADDING, so the ClientHello and JA4 are the same in both modes. TShark already handles this; the stateful mode makes our extractor match it.
- Damaged copies (baseline plus five kinds of damage, two pcaps): stateless mode matched TShark on 54 of 54 hellos, stateful mode on 42 of 54. The stateful mode gets 0/5 on reversed order and 4/5 on shuffled order, because a later packet arriving first is taken as the key source.

## Facts about the tools

- FoxIO tag v0.18.6 reports `ja4 0.18.5`; tag v1.0.1 reports `ja4 0.18.6` (its `rust/Cargo.toml` was not updated).
- FoxIO ja4 reads packets through TShark (`rtshark`), so TShark and FoxIO agreeing is two calculators on one packet parser.
- JA4T is not in TShark 4.4.19 or FoxIO v0.18.6; FoxIO v1.0.1 has it.

## Lab setup

- Track 1 captures: Docker (`cn-lab`, Debian) on a MacBook (aarch64), campus network. Versions in `results/versions.txt`.
- Track 2 runs and the L3 paired capture: Windows 11 with WSL2 Ubuntu (TShark 4.6.4, GCC 15.2.0, OpenSSL 3.5.5). Versions in `environment.txt` in each run folder.
