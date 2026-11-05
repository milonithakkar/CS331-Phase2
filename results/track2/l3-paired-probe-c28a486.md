# Earlier L3 probe (checker commit c28a486)

This was the first try at seeing one Docker connection from two capture points, before the full paired run in `results/l3_paired_batch/`. Miloni ran it on WSL and wrote down the results here; the pcaps from this probe were not kept.

- `container-eth0.pcap` had 27 packets and `wsl-eth0.pcap` had 80. The container's connection did not appear on WSL's `eth0` at all.
- Inside the container: client `172.17.0.3:42910` to `104.16.123.96:443`, SYN with MSS 1460, window scale 10, window 64240, TTL 64. The SYN-ACK had MSS 1460, window scale 7, window 29184, TTL 63.

What it showed: on Windows, Docker Desktop's traffic does not go through WSL's network interface, so WSL `eth0` is the wrong place to look. That is why the paired run then captured on the Windows side (the vEthernet adapter and the Wi-Fi card). That run is what T3 uses ([`docs/tests/T3_vantage_offload.md`](../../docs/tests/T3_vantage_offload.md)).
