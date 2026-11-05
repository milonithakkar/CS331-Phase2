# T7: Does ECH hide the site name? (L7)

Chromium visits the same site three ways (ECH off, ECH on, ECH with encrypted DNS) while we capture both TLS and DNS, and we check where the site name still shows.

## What we did

- Chromium opens `https://crypto.cloudflare.com/cdn-cgi/trace`. That page reports `sni=plaintext` or `sni=encrypted`, which is Cloudflare's own confirmation.
- Three runs:
  1. ECH off: Chrome policy `EncryptedClientHelloEnabled=false` (`captures/config/ech_off_policy.json`)
  2. ECH on: Chromium's default
  3. ECH with DoH: DNS over HTTPS to Cloudflare (`captures/config/doh_policy.json`)
- Capture filter `tcp port 443 or port 53`, so we see TLS and plain DNS.

## Questions

1. Is the real site name visible in the TLS SNI?
2. Is it visible in DNS?
3. Does JA4 change when ECH is on?

## Why

In Phase 1 we were asked about SNI and eSNI. SNI (RFC 6066 §3) sends the site name in plaintext, and telling which site an app talks to depends on it.

ECH (RFC 9849) encrypts the real ClientHello with a key published in DNS (the HTTPS record, RFC 9460). From outside you only see a shared public name. But DNS itself (RFC 1035) is plaintext unless you use DoH (RFC 8484).

## Control

Same browser, page and network. Only the ECH/DNS setting changes.

## Commands

```sh
make lab-capture CAPTURE_ID=tls_ech_off
make lab-capture CAPTURE_ID=tls_ech_on
make lab-capture CAPTURE_ID=tls_ech_doh
make lab-eval
cat results/track1/page-observations.md
```

## What we saw

| run | SNI on the wire | page says | DNS shows crypto.cloudflare.com? | JA4 (ja4-main) |
|---|---|---|---|---|
| ECH off | `crypto.cloudflare.com` | `sni=plaintext` | yes | `t13d1516h2_8daaf6152771_447d952d5833` |
| ECH on | `cloudflare-ech.com` | `sni=encrypted` | yes | `t13d1517h2_8daaf6152771_cb7bf5808d99` |
| ECH + DoH | - | page did not load | - | - |

- ECH adds one extension (0xfe0d), so JA4 goes from 16 to 17 extensions.
- With ECH on, the capture has 5 ClientHellos. 2 go to `cloudflare-ech.com`; the other 3 are Chromium's background connections (`redirector.gvt1.com`, `accounts.google.com`, `update.googleapis.com`). Plain DNS asks for `crypto.cloudflare.com` twice ([`recheck`](../../results/track1/recheck-20261106.md)).
- The DoH run is not a result. All 12 ClientHellos go to `cloudflare-dns.com`, the DoH server, and that name is itself looked up over plain DNS. On the campus network the DoH connection failed with a certificate error, so the page never loaded.

## What it means

- ECH hides the name in TLS, but plain DNS still gives it away. You need ECH and working encrypted DNS together.
- ECH also changes the browser's JA4, because of the extra extension.

## What it does not show

- The ECH + DoH case, which was blocked on our network.
- One site (Cloudflare) and one browser.
