# T2: Which fingerprints work without TLS keys? (L2)

Make one TLS 1.3 connection while saving its keys, then run ja4 on the same pcap twice: once without the keys and once with them.

## What we did

- curl makes 1 TLS 1.3, HTTP/1.1 request to `https://www.cloudflare.com/cdn-cgi/trace` with `SSLKEYLOGFILE` set, so it writes the session keys to a file.
- `ja4 pcap` (no keys) vs `ja4 --keylog-file keys pcap` (with keys), on the same pcap.

## Question

How many JA4X (certificate) and JA4H (HTTP) fingerprints come out without keys, and how many with keys?

## Why

A real network tap, a firewall or IDS, never has the keys. FoxIO lists JA4X and JA4H as fingerprints, so we check whether a tap can get them at all. In TLS 1.3 everything after the ServerHello is encrypted, including the certificate (RFC 8446 §2, §4.4.2), and the HTTP request is inside TLS too.

## Control

The same bytes, one pcap. The only change is whether the keys are given.

## Commands

```sh
make lab-capture CAPTURE_ID=tls_keylog_curl
make lab-eval      # columns ja4x_ja4h_no_keys and ja4x_ja4h_with_keys
```

## What we saw

| | JA4X + JA4H |
|---|---|
| no keys | 0 |
| with keys | 5 (4 JA4X, one per certificate in the chain, and 1 JA4H) |

JA4 and JA4S came out in both runs, since they come from the plaintext hellos.

## What it means

On TLS 1.3, JA4X and JA4H only work in a lab where you have the keys. A passive observer gets JA4 and JA4S, plus JA4T if the tool supports it.

## What it does not show

- TLS 1.2 sends the certificate in plaintext. We did not test TLS 1.2.
- The key log is not committed on purpose; you should never commit session keys. So the with-keys number cannot be rerun from the repository, only the no-keys one.
