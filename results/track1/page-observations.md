# ECH page outputs

Copied from Yash's saved page outputs for the T7 runs. The full outputs were not committed because they also contain the machine's IP address and other network details.

| Capture | Page field | Value |
|---|---|---|
| `tls_ech_off` | `h` | `crypto.cloudflare.com` |
| `tls_ech_off` | `sni` | `plaintext` |
| `tls_ech_on` | `h` | `crypto.cloudflare.com` |
| `tls_ech_on` | `sni` | `encrypted` |
| `tls_ech_doh` | page output | empty, the page did not load |

The DoH capture still has 12 ClientHellos, but they all go to the DoH server `cloudflare-dns.com`, so they do not show the page loading. The DoH case is not a result.
