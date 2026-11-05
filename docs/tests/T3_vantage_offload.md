# T3: Does where and how we capture change what we see? (L3)

Capture the same ClientHello with the kernel's segmentation offload on and then off, and compare. Then capture one connection at two points at the same time: inside the container and on the laptop's Wi-Fi card.

## What we did

- Mac lab: link MTU set to 1500. curl sends a large post-quantum ClientHello, bigger than one packet, 3 times with TSO/GSO on and 3 times with it off (`ethtool -K eth0 tso off gso off`). We also read the SYN and SYN-ACK values.
- Windows lab (Miloni, `results/l3_paired_batch/`): three requests from a Docker container, captured at the same time inside the container (`eth0`) and on the Wi-Fi card.

## Questions

1. In how many frames does the ClientHello appear, and how big are they?
2. Does JA4 change?
3. Are the TCP values we see really the client's and the server's?

## Why

Sir, about another group's topic: "enable certain optimisations like GRO, LSO… just changing one parameter in the network stack can affect completely the way you have to fingerprint."

With TSO/GSO the kernel hands one big packet to the network card and the card cuts it up later. A capture taken before that point sees a packet that never exists on the wire.

TCP fingerprints (JA4T, JA4TS) read SYN values such as MSS (RFC 9293 §3.7.1). They describe whichever TCP stack is closest to the capture point. A NAT only rewrites addresses and lowers the TTL. A proxy that ends the TCP connection and opens a new one replaces the values completely.

## Control

Same client, same hello, same server, same MTU. Only the offload switch changes. In the Windows run, the same three requests are seen at both capture points.

## Commands

```sh
make lab-capture CAPTURE_ID=tls_offload_on
make lab-capture CAPTURE_ID=tls_offload_off
tshark -r captures/pcaps/tls_offload_on.pcap  -Y "tcp.dstport==443 && tcp.len>1000" -T fields -e frame.len -e tcp.len
tshark -r captures/pcaps/tls_offload_off.pcap -Y "tls.handshake.type==1" -T fields -e frame.len -e tcp.segment.count
tshark -r captures/pcaps/tls_offload_on.pcap  -Y "tcp.flags.syn==1" -T fields -e tcp.options.mss_val -e ip.ttl -e tcp.options.wscale.shift
```

## What we saw

From the pcaps, re-checked with TShark ([`results/track1/recheck-20261106.md`](../../results/track1/recheck-20261106.md)):

| | ClientHello as captured | JA4 |
|---|---|---|
| offload on | 1 frame of 1586 bytes, bigger than the 1500 MTU | `t13d3012h2_1d37bd780c83_7f037f8d6676` |
| offload off | 2 frames, 1514 + 138 bytes | same |

The `biggest_frame` column in `summary.md` is the biggest frame in the whole pcap in either direction, which is why it shows 4913 and 2962 instead of 1586.

SYN values in the Mac capture:

| packet | MSS | TTL | window scale |
|---|---|---|---|
| our SYN | 1460 | 64 | 10 |
| SYN-ACK, supposedly from Cloudflare | 65495 | 63 | 7 |

65495 is the MSS of a 65535-byte loopback-style interface, not what Cloudflare sends. In the Windows run we could see both sides: on the Wi-Fi card Cloudflare's SYN-ACK had MSS 1400 and window scale 13, while inside the container the SYN-ACK had MSS 1460 and window scale 7. So the SYN-ACK the container sees is made by Docker's own network stack, not by the server.

Client SYN for the same request in the Windows run:

| where | TTL | window | window scale | MSS | timestamps |
|---|---|---|---|---|---|
| inside the container | 64 | 64240 | 10 | 1460 | yes (Linux) |
| on the Wi-Fi card | 128 | 65535 | 8 | 1460 | no (Windows) |

All four values that differ are Windows defaults, and the sequence numbers differ too. The connection leaving the laptop is opened by Windows, not passed through from the container. For comparison, traffic from a Linux VM on the `vEthernet (WSL)` adapter keeps its Linux values on the Wi-Fi with only the TTL going from 64 to 63, which is what a normal NAT does.

## What it means

- TLS fingerprints (JA4) are not affected. Packet sizes and counts depend on how the capture was set up.
- TCP fingerprints describe the nearest TCP stack. Here a JA4TS taken in the container would describe Docker, and a JA4T taken on the Wi-Fi would describe Windows, not the Linux container.

## What it does not show

- The Mac run has one capture point, inside the container. The Windows run adds the Wi-Fi card, but only 3 requests.
- We did not capture on a physical wire outside the laptop, and the capture on the Docker bridge failed.
- JA4T is not in our pinned tools (TShark 4.4.19, FoxIO v0.18.6). FoxIO v1.0.1 has it; on `tls_offload_on` it gives `64240_2-4-8-1-3_1460_10`, the container's own SYN.
