# JA4: client TLS/QUIC fingerprint

Source: FoxIO JA4+ [v1.0.1](https://github.com/FoxIO-LLC/ja4/releases/tag/v1.0.1) (tag commit `896b951`), [technical details](https://github.com/FoxIO-LLC/ja4/blob/v1.0.1/technical_details/README.md), [Python source](https://github.com/FoxIO-LLC/ja4/blob/v1.0.1/python/ja4.py). Not an IETF standard.

| | |
|---|---|
| Message | Client ClientHello, over TCP (`t`) or QUIC (`q`). |
| Fields | Transport, TLS version, whether SNI is present, number of ciphers, number of extensions, first ALPN value, the cipher list, the extension list, the signature algorithms. |
| How they are combined | Three parts, `a_b_c`. Part a is readable, e.g. `t13d1517h2` = TCP, TLS 1.3, SNI present, 15 ciphers, 17 extensions, HTTP/2. Part b is a hash of the sorted cipher list. Part c is a hash of the sorted extension list plus the signature algorithms. GREASE is removed. |
| Output | Readable prefix plus two 12-character SHA-256 pieces. |
| Visible without keys? | Yes for a normal ClientHello. With ECH only the outer one. For QUIC v1 the Initial keys can be computed by anyone (RFC 9001 §5.2), so the ClientHello can be recovered too. |
| RFCs for the fields | ClientHello: [RFC 8446 §4.1.2](https://www.rfc-editor.org/rfc/rfc8446.html#section-4.1.2). SNI: [RFC 6066 §3](https://www.rfc-editor.org/rfc/rfc6066.html#section-3). ALPN: [RFC 7301](https://www.rfc-editor.org/rfc/rfc7301.html). Signature algorithms: [RFC 8446 §4.2.3](https://www.rfc-editor.org/rfc/rfc8446.html#section-4.2.3). GREASE: [RFC 8701](https://www.rfc-editor.org/rfc/rfc8701.html). QUIC: [RFC 9001](https://www.rfc-editor.org/rfc/rfc9001.html). |
| Weak spot | Sorting fixes the shuffle problem of JA3, but tools do not agree on GREASE in the signature algorithms. TShark 4.6.8 and FoxIO v0.18.6 keep it, FoxIO v1.0.1 removes it, so the same 50 Chromium hellos give 15 JA4s or 1 (T5). |

Our C++ extractor outputs the ClientHello bytes and does not compute JA4 itself. FoxIO's Rust tool reads packets through TShark, so the two agreeing is not two independent parsers.
