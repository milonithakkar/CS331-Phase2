# Captures

- `pcaps/` has the 16 pcaps: one early QUIC baseline capture ([`docs/quic-baseline.md`](../docs/quic-baseline.md)) and the 15 Track 1 captures from Yash's Docker lab.
- 14 of the Track 1 pcaps have packets. `tls_ipv6.pcap` is an empty 24-byte pcap header from the IPv6 attempt that failed (no IPv6 route). We keep it as a record of the attempt, not as a result.
- `manifest.json` lists every capture with its SHA-256 and its metadata file in `metadata/`. `make check` checks them.
- `config/` has the two Chromium policy files used in T7.
- `keylogs/` is ignored by git. TLS key logs must never be committed, and none were.
- `make lab-capture CAPTURE_ID=<id>` writes new captures to `captures/reproduced/`, which is also ignored. Check the packet count, hash and contents before adding one to the manifest.

What each capture is for is in [`docs/tests/`](../docs/tests/README.md). Where the Track 1 captures came from and what was checked is in [`docs/track1-evidence-audit.md`](../docs/track1-evidence-audit.md).
