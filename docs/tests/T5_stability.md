# T5: Does one browser give one fingerprint? (L5)

Open many new connections from the same client and count how many different fingerprints come out. Do the same for two apps that use the same TLS library.

## What we did

- Chromium 156: 10 page loads, which gave 50 ClientHellos (the page plus Chromium's own background connections).
- curl: 10 connections. Python `urllib`: 10 connections. Both use OpenSSL 3.5.7.
- We counted distinct JA3 and JA4 values with:
  - our lab tools, FoxIO tag v0.18.6 and TShark 4.4.19,
  - FoxIO `main` at commit c6469a7, built as `ja4-main`,
  - later, FoxIO tag v1.0.1 and TShark 4.6.8.

## Questions

1. Does the same client always get the same fingerprint?
2. Do two apps on the same library get the same fingerprint?

## Why

Sir: "the same application cannot have different hash? … it can", and "will it work tomorrow?". A blocklist (his example: block Firefox, allow Chrome) only works if one client gives one fingerprint.

In Phase 1 our detector flagged real Chrome as fake because its JA3 changed. Chrome shuffles its extensions, which RFC 8446 §4.1.2 allows. Clients also insert GREASE values (RFC 8701) that fingerprints are supposed to ignore.

## Control

Same machine, same server, a fresh browser profile each time. curl and Python are the simple clients.

## Commands

```sh
make lab-capture CAPTURE_ID=tls_chromium_repeat
make lab-capture CAPTURE_ID=tls_curl_repeat
make lab-capture CAPTURE_ID=tls_python_repeat
make lab-eval
# with a newer TShark:
tshark -r captures/pcaps/tls_chromium_repeat.pcap -Y "tls.handshake.type==1" -T fields -e tls.handshake.ja4 | sort -u | wc -l
# first signature algorithm of each hello:
tshark -r captures/pcaps/tls_chromium_repeat.pcap -Y "tls.handshake.type==1" -T fields -e tls.handshake.ja4_r | cut -d_ -f4 | cut -d, -f1 | sort -u
```

## What we saw

| client | hellos | distinct JA3 | distinct JA4 (TShark 4.4.19, FoxIO v0.18.6) | distinct JA4 (FoxIO main) |
|---|---|---|---|---|
| Chromium 156 | 50 | 50 | 15 | 1 |
| curl | 10 | 1 | 1 | 1 |
| Python | 10 | 1 | 1 | 1 |

Re-checked on 6 November with more tool versions on the same pcap ([`results/track1/recheck-20261106.md`](../../results/track1/recheck-20261106.md)):

| tool | distinct JA4 in Chromium's 50 hellos |
|---|---|
| TShark 4.4.19 and TShark 4.6.8 | 15 |
| FoxIO tag v0.18.6 | 15 |
| FoxIO tag v1.0.1 (31 Aug 2026) and FoxIO main | 1 |

Why 15: Chromium puts a random GREASE value first in its signature algorithms. The 50 hellos use 15 different GREASE values there (`0a0a` to `fafa`), one per JA4. The JA4 spec says to ignore GREASE. FoxIO fixed this in commit d66336e ("ignore grease in sigalgs"), first released in v1.0.1. TShark 4.6.8 still counts it.

Same library: curl and Python share JA4 part b (`1d37bd780c83`, OpenSSL's cipher list) but differ in parts a and c, because curl uses HTTP/2 and 13 extensions and Python uses HTTP/1.1 and 14 extensions (it adds session_ticket). So they share part of the fingerprint, not all of it.

## What it means

- JA3 does not work for Chromium: every connection gave a new JA3.
- The JA4 of one browser depends on which tool and version computed it: 15 values with TShark or FoxIO v0.18.6, 1 with FoxIO v1.0.1. A JA4 list built with one tool will miss most Chromium connections seen by another.
- A JA4 database should store the tool and version with each fingerprint, and fingerprints should be recomputed after a tool update.
- The Chrome JA4 we wrote down in Phase 1 (`..._541cd5a3d78e`) is one of these 15.

## What it does not show

- One Chromium version on one OS. Firefox was not tested. The 50 hellos include background connections.
- This is not a new bug: FoxIO already fixed it. What we show is that tools in use today, current TShark and older FoxIO, still give different answers.
