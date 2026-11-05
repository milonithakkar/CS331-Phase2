# Early QUIC baseline capture

Before the Track 1 captures existed, we checked that TShark and FoxIO agree on one real QUIC v1 capture. This is that check. It is separate from the QUIC captures used in T1.

## The capture

`captures/pcaps/quic-v1-baseline.pcapng`, SHA-256 `1cd5f8c05b5b966eed1ae339eeec80b86b830deccda81489dabefb28f272ec7a`. One aioquic 1.7.2 client requesting `https://cloudflare.com` over HTTP/3, captured on 3 November 2026. The file has 27 packets and no decryption secrets inside it.

## What the tools gave

Both TShark 4.6.4 and FoxIO ja4 (built from tag v1.0.1, tag commit `896b951`; the binary reports 0.18.6) gave the same JA4 for the ClientHello in frame 1:

```text
q13d0307h3_55b375c5d22e_1cecd519fee8
```

TShark also gave JA3 `121f6e4f22ee3205214b0676ed64b9e2`. The commands are in `captures/metadata/quic-v1-baseline.json` and the outputs, unchanged, in `results/baseline/`.

## What it shows

On this one capture both tools read the QUIC ClientHello and agree. Since FoxIO reads packets through TShark, this is one parser checked twice, not two independent ones. T1 later tested this properly with two clients and five connections each.

Not kept from this run: the exact aioquic script, the exact capture command, and a record linking the ja4 binary to the source tag. To rerun the tools on this file, see [`baseline-procedure.md`](baseline-procedure.md).
