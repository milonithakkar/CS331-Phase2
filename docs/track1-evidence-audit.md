# Track 1 data: where it came from and what we checked

## Where it came from

Yash made the Track 1 captures in his Docker lab and they were copied into this repository from his repository at commit [`b1ffd7c`](https://github.com/YashG1836/CS331-T013-TLS_Fingerprint_2/commit/b1ffd7c65ae47af1b62184549262ef838c4a234d). That gave 15 pcaps, one raw JSON output per pcap (`results/raw/`), and his summary. His summary is kept unchanged in `results/imported/`; `results/summary.md` and `.json` are our checked copy.

All 15 pcaps were checked when they were copied in. 14 have packets. `tls_ipv6.pcap` is only a 24-byte pcap header with no packets. Every pcap's SHA-256 is in `captures/manifest.json` and is checked by `make check`.

## What we changed

- IPv6: the original summary said `tools_agree: true` for the empty IPv6 capture, because two empty results compare equal. Our copy marks it as not comparable.

## What we re-checked on 6 November

Some numbers in the test write-ups were not kept in the raw JSON, so on 6 November we ran TShark and FoxIO again directly on the committed pcaps ([`results/track1/recheck-20261106.md`](../results/track1/recheck-20261106.md)):

- L3: the ClientHello is 1 frame of 1586 bytes with offload on and 1514 + 138 bytes with it off; the SYN-ACK in the container has MSS 65495. The `biggest_frame` column in the summary is the biggest frame of the whole pcap, which is a different number.
- L5: 15 distinct JA4s with TShark 4.4.19, TShark 4.6.8 and FoxIO v0.18.6; 1 with FoxIO v1.0.1 and FoxIO main. The 15 come from 15 different GREASE values at the start of the signature algorithms.
- L7: with ECH on, plain DNS still asks for `crypto.cloudflare.com`. In the DoH run all 12 ClientHellos go to the DoH server and the page did not load.

## What cannot be rerun from the repository

- L2 with keys: the key log is not committed on purpose. The no-key half can be rerun; the with-keys number (5) cannot.
- L6: there are no IPv6 packets to analyse.

## Things to keep in mind

- FoxIO's ja4 reads packets through TShark, so TShark and FoxIO agreeing is one parser checked twice.
- The two labs had different tool versions. Yash's Docker lab: TShark 4.4.19, FoxIO tag v0.18.6 (binary reports 0.18.5), FoxIO main c6469a7 (reports 0.18.6), see `results/versions.txt`. The WSL machine: TShark 4.6.4, see `results/preflight.json`. Do not mix numbers from the two without saying which.
- The early aioquic capture (`quic-v1-baseline.pcapng`) and the T1 captures are different clients to different sites, so their JA4s are expected to differ.
- Key logs and personal meeting notes were not copied in. The ECH page outputs in `results/track1/page-observations.md` were copied without the machine's IP address.

## Rerunning

```sh
make lab-image
make lab-eval                               # recompute from the committed pcaps, writes to results/reproduced/
make lab-capture CAPTURE_ID=tls_ipv6        # new capture, needs network, writes to captures/reproduced/
```

A new capture is not added to the manifest automatically. Check its packet count and contents before adding it, and keep key logs out of the repository.
