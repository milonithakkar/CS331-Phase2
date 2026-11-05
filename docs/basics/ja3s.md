# JA3S: server TLS fingerprint

Source: [Salesforce JA3](https://github.com/salesforce/ja3). Not an IETF standard.

| | |
|---|---|
| Message | Server ServerHello. |
| Fields | TLS version, the cipher the server picked, extension types. |
| How they are combined | Comma-joined, extension order kept, GREASE removed, then MD5. |
| Output | 32 hex characters. |
| Visible without keys? | Yes. In TLS 1.3 everything after the ServerHello is encrypted, but the ServerHello itself is not. |
| RFCs for the fields | ServerHello: [RFC 5246 §7.4.1.3](https://www.rfc-editor.org/rfc/rfc5246.html#section-7.4.1.3), [RFC 8446 §4.1.3](https://www.rfc-editor.org/rfc/rfc8446.html#section-4.1.3). GREASE: [RFC 8701](https://www.rfc-editor.org/rfc/rfc8701.html). |
| Weak spot | It depends on what the client offered, not only on the server. If a proxy ends the TLS connection, JA3S describes the proxy. |
