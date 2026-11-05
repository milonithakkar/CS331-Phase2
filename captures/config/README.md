# Chromium policies for T7

- `ech_off_policy.json` turns ECH off (`EncryptedClientHelloEnabled=false`).
- `doh_policy.json` turns on DNS over HTTPS to Cloudflare.

The capture recipe copies them into the lab container only, so they do not change the host machine. The page outputs from the ECH runs are in `results/track1/page-observations.md`.
