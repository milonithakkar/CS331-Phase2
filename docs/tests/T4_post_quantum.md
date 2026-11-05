# T4: What does a post-quantum key share change? (L4)

The same curl connects 5 times offering only the post-quantum hybrid group, and 5 times offering only the classic group. We compare the ClientHello size and the fingerprints.

## What we did

- `curl --curves X25519MLKEM768` (post-quantum hybrid), 5 times.
- `curl --curves X25519` (classic), 5 times.
- Same server; TShark and ja4 read both pcaps.

## Questions

1. How big is the ClientHello?
2. Which group did the server actually pick?
3. Do JA3 and JA4 change, and why?

## Why

Sir: "Now we have PQC… nobody knows whether the fingerprint will work or not."

Post-quantum keys are large. X25519MLKEM768 makes the key share 1216 bytes instead of 32 (RFC 8446 §4.2.8 for the key_share extension). The ClientHello then no longer fits in one packet. JA4 has no field for groups, so it might not change at all.

## Control

Same curl binary, same server, everything the same except `--curves`.

## Commands

```sh
make lab-capture CAPTURE_ID=tls_pq_on
make lab-capture CAPTURE_ID=tls_pq_off
make lab-eval
```

## What we saw

| | ClientHello | group picked by the server | JA4 |
|---|---|---|---|
| PQ on | 1511 bytes | 4588 (0x11EC, X25519MLKEM768) | `t13d3012h2_1d37bd780c83_7f037f8d6676` |
| PQ off | 508 bytes | 29 (x25519) | `t13d3014h2_1d37bd780c83_3eb322dd144c` |

JA4 changed anyway: 12 extensions vs 14. The `ja4_r` output in `results/raw/` shows why:

- With only the PQ group, OpenSSL leaves out `ec_point_formats` (0x000b, RFC 8422 §5.1.2).
- The small classic hello (508 bytes) gets the `padding` extension (0x0015, RFC 7685), which OpenSSL adds to hellos between 256 and 511 bytes.

curl's default hello, with both groups, has 13 extensions (T6), between the two.

## What it means

- A crypto setting changes the fingerprint through side effects, not through a group field. A JA4 list has to be rebuilt as clients move to post-quantum key shares.
- The hello no longer fits in one packet, which breaks anything that assumes one packet holds one hello. How it shows up in a capture depends on offload (T3).

## What it does not show

- Only OpenSSL through curl. Chromium also uses group 4588 by default (T1, T5), but we did not turn PQ off in Chromium.
