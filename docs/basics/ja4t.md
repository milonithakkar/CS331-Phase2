# JA4T: TCP SYN fingerprint

Source: FoxIO JA4+ [v1.0.1](https://github.com/FoxIO-LLC/ja4/releases/tag/v1.0.1), [Rust TCP code](https://github.com/FoxIO-LLC/ja4/blob/v1.0.1/rust/ja4/src/tcp.rs). Not an IETF standard.

| | |
|---|---|
| Message | The client's first TCP SYN, before any TLS. (JA4TS is the same for the SYN-ACK.) |
| Fields | TCP window size, the list of TCP option kinds in order, MSS, window scale. |
| How they are combined | Not hashed. Written out as `window_options_mss_wscale`, e.g. `64240_2-4-8-1-3_1460_10`. |
| Visible without keys? | Yes, it is below TLS. But it depends on where you capture. |
| RFCs for the fields | TCP and its options: [RFC 9293](https://www.rfc-editor.org/rfc/rfc9293.html). MSS: [RFC 6691](https://www.rfc-editor.org/rfc/rfc6691.html). |
| Weak spot | It describes the TCP stack nearest to the capture point. A NAT keeps it, but a proxy that ends the TCP connection and opens a new one replaces it. |

Tool support: not in TShark 4.4.19 or FoxIO v0.18.6; FoxIO v1.0.1 has it. In T3, the same container connection had Linux SYN values inside the container (TTL 64, window 64240, wscale 10, timestamps) and Windows values on the Wi-Fi card (TTL 128, window 65535, wscale 8, no timestamps). A JA4T taken on the Wi-Fi would describe Windows, not the container.
