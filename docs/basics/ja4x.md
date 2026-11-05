# JA4X: X.509 certificate fingerprint

Source: FoxIO JA4+ [v1.0.1](https://github.com/FoxIO-LLC/ja4/releases/tag/v1.0.1), [Python code](https://github.com/FoxIO-LLC/ja4/blob/v1.0.1/python/ja4x.py). Not an IETF standard.

| | |
|---|---|
| Object | Each certificate in the server's certificate chain. |
| Fields | The issuer and subject name fields, and the list of certificate extension OIDs. |
| How they are combined | Three truncated SHA-256 hashes, one per part. It describes how the certificate was built, not who it belongs to. |
| Visible without keys? | TLS 1.2: yes, the certificate is sent in plaintext. TLS 1.3: no, the certificate is encrypted after the ServerHello. |
| RFCs for the fields | X.509: [RFC 5280](https://www.rfc-editor.org/rfc/rfc5280.html). TLS 1.3 Certificate message: [RFC 8446 §4.4.2](https://www.rfc-editor.org/rfc/rfc8446.html#section-4.4.2). |
| Weak spot | On TLS 1.3 a passive tap never sees it. |

In T2 (one TLS 1.3 connection) FoxIO gave 0 JA4X without the key log and 4 with it, one per certificate in the chain. The key log is not committed, so only the no-key half can be rerun from the repository.
