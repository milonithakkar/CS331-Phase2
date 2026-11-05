# Tests

In Phase 1 our tests did not have a clear question: one capture per client, and then we looked at what came out. This time each test asks one question and changes one thing.

How each test is set up:

1. The question is written down before capturing.
2. Only one thing changes. The client, server and tools stay the same.
3. There is a control, the normal case, next to the changed case.
4. Only real tools compute the fingerprints: TShark (Wireshark, C) and FoxIO ja4 (Rust).
5. The numbers come from `results/summary.md`. Numbers the summary does not keep were re-checked on the committed pcaps in [`results/track1/recheck-20261106.md`](../../results/track1/recheck-20261106.md).
6. Each file ends with what the test does not show.

FoxIO's ja4 reads packets through TShark, so the two tools agreeing does not mean two independent parsers agree.

| File | Question | What changes | Result |
|---|---|---|---|
| [T0](T0_lab_checks.md) | Do the build, tests and capture hashes check out? | - | passed |
| [T1](T1_quic.md) | Can the tools fingerprint QUIC? | TCP to QUIC, two clients | 5/5 hellos in both tools; our C++ extractor gives the same bytes |
| [T2](T2_keys.md) | Which fingerprints work without TLS keys? | keys given or not | JA4X + JA4H: 0 without keys, 5 with keys |
| [T3](T3_vantage_offload.md) | Does where and how we capture change what we see? | offload on/off, capture point | 1 frame of 1586 B vs 1514 + 138 B; TCP values come from the nearest TCP stack |
| [T4](T4_post_quantum.md) | What does a post-quantum key share change? | PQ group on/off | 1511 B vs 508 B; JA4 has 12 vs 14 extensions |
| [T5](T5_stability.md) | Does one browser give one fingerprint? | 50 connections from one Chromium | 50 JA3s; 15 JA4s or 1, depending on the tool version |
| [T6](T6_ipv6.md) | Does IPv6 change the fingerprint? | IPv4 vs IPv6 | not measured, no IPv6 route |
| [T7](T7_ech_dns.md) | Does ECH hide the site name? | ECH off, on, on + DoH | hidden in TLS, still in plain DNS; DoH run did not load |

Certificate compression and abridged certificates were only read about, not tested (see `LEDGER.md`).

To rerun:

```sh
make lab-image                            # build the Docker lab once
make lab-eval                             # recompute every number from the committed pcaps, no internet needed
make lab-capture CAPTURE_ID=tls_pq_on     # make a test's capture again (needs internet)
```
