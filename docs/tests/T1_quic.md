# T1: Can the tools fingerprint QUIC? (L1)

Load the same small page over QUIC with two different clients and check whether TShark and FoxIO ja4 recover the ClientHello.

## What we did

- Chromium 156 loads `https://cloudflare-quic.com/cdn-cgi/trace` over HTTP/3, 5 times.
- curl 8.14.1 (HTTP/3) loads the same page, 5 times.
- tcpdump records UDP port 443, then TShark and ja4 read the pcap.

## Questions

1. Does each tool find every QUIC ClientHello (5 of 5)?
2. Do the two tools give the same JA4?
3. Does this still work when the ClientHello is split into many pieces?

## Why

Sir said: "if I have to give you an objective, I would say, do QUIC."

In QUIC the ClientHello is not in plaintext. It sits inside encrypted Initial packets. To read it a tool has to:

1. derive the Initial keys from a public salt and the Destination Connection ID (RFC 9001 §5.2),
2. remove header protection (RFC 9001 §5.4),
3. decrypt the packet (AES-128-GCM),
4. put the CRYPTO frames back together by offset (RFC 9000 §19.6).

We expected at least one tool to get this wrong.

## Control

Same server, page and network; only the client changes. The TCP captures in T5 are the normal case.

## Commands

```sh
make lab-capture CAPTURE_ID=quic_chromium
make lab-capture CAPTURE_ID=quic_curl
make lab-eval
```

## What we saw

| capture | hellos (TShark / ja4) | biggest hello | CRYPTO pieces | same JA4 in both tools? |
|---|---|---|---|---|
| quic_chromium | 5 / 5 | 1887 bytes | 21 | yes |
| quic_curl | 5 / 5 | 1505 bytes | 1 | yes |

- The server picked group 4588 (X25519MLKEM768) in both.
- Chromium used ECH inside QUIC by default; the outer name was `cloudflare-ech.com`.

## What it means

- Our hypothesis was wrong. TShark 4.4 and FoxIO 0.18 handle complete QUIC captures, including Chromium's hello in 21 pieces.
- QUIC Initial encryption does not hide the ClientHello from a passive observer, because the keys come from the connection ID, which is in the packet.
- Our own C++ extractor (Track 2) gets the same 5 ClientHellos per capture, byte for byte equal to TShark. See [`docs/quic/implementation-status.md`](../quic/implementation-status.md).

## What it does not show

- Connection migration: after the client changes network, the new path has no ClientHello at all (RFC 9000 §9). Not tested.
- A capture that misses the first packet, Retry, QUIC v2 (RFC 9369): not tested.
- Only two clients and one server.
