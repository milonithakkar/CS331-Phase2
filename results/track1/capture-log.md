# Track 1 capture log

The capture log is from Yash's Docker lab on macOS (aarch64), with tcpdump on the container's `eth0`. It lists the packet count of each saved pcap and shows each client command repeated as many times as the recipe says. The full commands, filters and repeat counts are in `captures/manifest.json` and `captures/metadata/<capture_id>.json`. The machine's file path was left out.

The log does not record whether each request succeeded. For IPv6 it shows zero packets, matching the empty `tls_ipv6.pcap`. All captures were taken inside Docker Desktop, not on a physical network cable, which matters for the packet sizes and SYN values in T3.
