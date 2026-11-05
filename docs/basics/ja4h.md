# JA4H: HTTP request fingerprint

Source: FoxIO JA4+ [v1.0.1](https://github.com/FoxIO-LLC/ja4/releases/tag/v1.0.1), [Python code](https://github.com/FoxIO-LLC/ja4/blob/v1.0.1/python/ja4h.py). Not an IETF standard.

| | |
|---|---|
| Message | The HTTP request (HTTP/1.x or HTTP/2). |
| Fields | Method, HTTP version, whether there is a cookie or referer, number of headers, language, the header names, the cookie names and values. |
| How they are combined | A readable prefix plus hashes of the header names and of the sorted cookie fields. |
| Visible without keys? | Only for plain HTTP. For HTTPS the request is inside TLS, so you need the keys. For HTTP/3 you need the QUIC 1-RTT keys; the public Initial keys are not enough. |
| RFCs for the fields | HTTP: [RFC 9110](https://www.rfc-editor.org/rfc/rfc9110.html). HTTP/2: [RFC 9113](https://www.rfc-editor.org/rfc/rfc9113.html). HTTP/3: [RFC 9114](https://www.rfc-editor.org/rfc/rfc9114.html). Cookies: [RFC 6265](https://www.rfc-editor.org/rfc/rfc6265.html). |
| Weak spot | A passive tap on HTTPS never sees it. It also changes from request to request. |

In T2 FoxIO gave 0 JA4H without the key log and 1 with it.
