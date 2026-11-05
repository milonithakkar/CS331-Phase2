# JA4S: server TLS/QUIC fingerprint

Source: FoxIO JA4+ [v1.0.1](https://github.com/FoxIO-LLC/ja4/releases/tag/v1.0.1), [technical details](https://github.com/FoxIO-LLC/ja4/blob/v1.0.1/technical_details/README.md). Not an IETF standard.

| | |
|---|---|
| Message | Server ServerHello, over TCP or QUIC. |
| Fields | Transport, TLS version, number of extensions, ALPN, the cipher the server picked, the extension list. |
| How they are combined | Readable prefix, the chosen cipher, and a hash of the extension list in the order sent. Unlike JA4, this list is not sorted. |
| Visible without keys? | Yes over TCP. Over QUIC the ServerHello is in the server's Initial packets, which use the same public Initial keys. |
| RFCs for the fields | [RFC 8446 §4.1.3](https://www.rfc-editor.org/rfc/rfc8446.html#section-4.1.3), [RFC 7301](https://www.rfc-editor.org/rfc/rfc7301.html), [RFC 8701](https://www.rfc-editor.org/rfc/rfc8701.html). |
| Weak spot | Like JA3S, it changes with what the client offered, and describes whatever ended the TLS connection. |

Example from T2 (curl to www.cloudflare.com): `t130200_1302_234ea6891581`. We did not study JA4S further.
