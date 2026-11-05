# What the results show

A summary of the WSL verification runs and the Track 1 data, in one place. Details are in [`docs/quic/implementation-status.md`](../docs/quic/implementation-status.md), [`docs/verification-batch.md`](../docs/verification-batch.md) and [`docs/tests/`](../docs/tests/README.md).

## Environment

All three WSL runs: Ubuntu on WSL2 (kernel 6.18.33.2), TShark 4.6.4, GCC 15.2.0, OpenSSL 3.5.5, Python 3.14.4. The two earlier runs had pytest 39/39; the 5 November run (`verification-batch-20261105-105416`) had CTest 1/1 and pytest 52/52, after more tests were added. `make check`: 16 captures, 15 with packets and the empty IPv6 one.

## Track 2: C++ extractor vs TShark

Captured order, 5 November run:

| capture | client Initials | stateless: decrypted / failed | stateful: decrypted / failed | server Initials decrypted | ClientHellos equal to TShark |
|---|---|---|---|---|---|
| curl | 20 | 10 / 10 | 20 / 0 | 5 / 5 | 5 / 5 |
| Chromium | 16 | 10 / 6 | 16 / 0 | 5 / 5 | 5 / 5 |

The failing packets in stateless mode are the client Initials sent to the server's connection ID; the header inventory (`track2/quic-initial-dcid-inventory.json`) shows them in each of the five flows per capture. The stateful mode decrypts them, but they only carry ACK and PADDING, so both modes find the same 5 ClientHellos.

Damaged copies, ClientHellos equal to TShark on the same damaged file (same for curl and Chromium):

| damage | stateless | stateful |
|---|---|---|
| none | 5 / 5 | 5 / 5 |
| reversed order | 5 / 5 | 0 / 5 |
| shuffled order (seed 331) | 5 / 5 | 4 / 5 |
| one CRYPTO packet dropped | 5 / 5 | 5 / 5 |
| every third CRYPTO packet dropped | 2 / 2 | 2 / 2 |
| one CRYPTO packet cut by 64 bytes | 5 / 5 | 5 / 5 |
| total | 27 / 27 | 21 / 27 |

So the stateful mode matches TShark on the captured order but loses ClientHellos when packets are reordered. TShark is the comparison, not an independent truth: when it finds only 2 ClientHellos, we can only check those 2.

## L3: SYN values

- Mac lab, both offload pcaps: every client SYN is MSS 1460 / window scale 10 / window 64240 / TTL 64, every SYN-ACK 65495 / 7 / 65408 / 63.
- Windows run (`l3_paired_batch/`): three requests captured inside the container and on the Wi-Fi card. Inside the container the SYN has Linux values (TTL 64, window 64240, window scale 10, timestamps) and the SYN-ACK has MSS 1460 / window scale 7. On the Wi-Fi card the SYN has Windows values (TTL 128, window 65535, window scale 8, no timestamps), the sequence numbers differ, and Cloudflare's SYN-ACK has MSS 1400 / window scale 13. The capture on the Docker bridge failed, and the Wi-Fi capture is on the laptop's adapter, not on a separate tap.

What this means is written up in [`docs/tests/T3_vantage_offload.md`](../docs/tests/T3_vantage_offload.md).

## Seven-row checker

L1, L2, L3, L4, L5 and L7 pass; L6 is not measured. The optional IPv6 loopback capture got no packets, so the live part reports FAIL. The WSL machine has no global IPv6 route.

## Track 1 points to keep in mind

- L2: the with-keys number (5) cannot be rerun, because the key log is not committed.
- L4: 1511 vs 508 bytes, groups 4588 vs 29. JA4 changes through the extension list, not through the group.
- L5: 50 hellos, 50 JA3s, 15 JA4s with TShark and FoxIO v0.18.6. The count of 1 with FoxIO main and v1.0.1 was re-checked on 6 November (`track1/recheck-20261106.md`).
- L6: the IPv6 pcap is empty.
- L7: in each ECH capture, 2 of the 5 ClientHellos are for the target (`crypto.cloudflare.com` with ECH off, `cloudflare-ech.com` with ECH on); the rest are Chromium background connections. The DoH run did not load.
- FoxIO's ja4 reads packets through TShark, so the two agreeing is one parser checked twice.
