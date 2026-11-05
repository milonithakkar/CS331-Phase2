# T6: Does IPv6 change the fingerprint? (L6)

The same curl connects over IPv4 and over IPv6, and we compare fingerprints.

## What we did

- `curl -4` to `https://www.cloudflare.com/cdn-cgi/trace`, 3 times.
- `curl -6` to the same URL, 3 times.

## Question

Does JA4 stay the same across IP versions, and which TCP/IP values change?

## Why

Sir: "how IPv6 comes right now… are they going to work, potentially the fingerprinting approach completely changes."

JA4 has no IP field, so we expected it to stay the same. But:

- the IPv6 header is 40 bytes instead of 20, so MSS is usually 1440 instead of 1460, which changes JA4T,
- IPv6 privacy addresses change over time (RFC 8981 §3.3), which breaks tracking by IP address,
- IPv6 has a different header chain (RFC 8200 §4).

## Control

The same curl and server, back to back. Only `-4` or `-6` changes.

## Commands

```sh
make lab-capture CAPTURE_ID=tls_ipv4
make lab-capture CAPTURE_ID=tls_ipv6
make lab-eval
```

## What we saw

- IPv4: 3 hellos, JA4 `t13d3013h2_1d37bd780c83_8537cf56674e`.
- IPv6: empty capture. Neither the container nor the Mac had a global IPv6 address on the campus network, and the WSL machine had no IPv6 route either, so `curl -6` failed.

## Status: not measured

`tls_ipv6.pcap` is a 24-byte pcap header with no packets. The original summary said `tools_agree: true` because two empty results compare equal; our `results/summary.*` marks it as not comparable. We claim no IPv6 result.

To try again on a network that has IPv6, use `code/scripts/capture_ipv6_egress.sh` (see [`docs/verification-batch.md`](../verification-batch.md)) or `make lab-capture CAPTURE_ID=tls_ipv6`.
