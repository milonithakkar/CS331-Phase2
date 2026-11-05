# JA3: client TLS fingerprint

Source: [Salesforce JA3](https://github.com/salesforce/ja3). Not an IETF standard.

| | |
|---|---|
| Message | Client ClientHello, normally over TCP. Wireshark can also compute it for a ClientHello carried in QUIC. |
| Fields | TLS version, cipher suites, extension types, supported groups, EC point formats. |
| How they are combined | The five fields are joined with commas; values inside a field are joined with hyphens. The order is kept as sent. GREASE values are removed. |
| Output | MD5 of that string, 32 hex characters. |
| Visible without keys? | Yes, for a normal ClientHello. With ECH only the outer ClientHello is visible. |
| RFCs for the fields | ClientHello: [RFC 5246 §7.4.1.2](https://www.rfc-editor.org/rfc/rfc5246.html#section-7.4.1.2), [RFC 8446 §4.1.2](https://www.rfc-editor.org/rfc/rfc8446.html#section-4.1.2). Supported groups: [RFC 8422 §5.1](https://www.rfc-editor.org/rfc/rfc8422.html#section-5.1). GREASE: [RFC 8701](https://www.rfc-editor.org/rfc/rfc8701.html). |
| Weak spot | Because the order is kept, a client that shuffles its extensions gets a new JA3 on almost every connection. In T5, Chromium gave 50 JA3s in 50 connections. |
