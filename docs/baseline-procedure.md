# How to record a reference-tool baseline

Steps for running TShark and FoxIO ja4 on a capture and keeping the result in a form we can check later. The first capture done this way is described in [`quic-baseline.md`](quic-baseline.md).

## Tools

- FoxIO ja4 built from tag v1.0.1 (tag commit `896b951e438c2979e0ef077a94ad390b1c973c23`). Its binary reports version 0.18.6, so write down both.
- TShark 4.0.6 or later (FoxIO's Rust tool needs it, because it reads packets through TShark).
- `results/preflight.json` lists the versions on the WSL machine used on 3 November: TShark 4.6.4, OpenSSL 3.5.5, CMake 4.2.3, GCC 15.2.0, Python 3.14.4, tcpdump 4.99.6.

## Steps

1. Make one controlled capture with no personal browsing, passwords or key logs in it.
2. Write the interface, capture filter, start and stop time, tool versions, server address and client command in `captures/metadata/<id>.json`.
3. Add it to `captures/manifest.json` with its SHA-256 and the metadata path.
4. Run ja4 and TShark on the same file and keep the exact commands and outputs.
5. Run `make check` to check the hash and the metadata.
6. To produce a fresh machine-readable run with the repository's own runner:

   ```sh
   make baseline CAPTURE_ID=<registered-id>
   ```

   This writes the capture hash, tool versions, exact commands, exit codes and raw outputs under `results/reference/`. Its commands are not the same as the ones used by hand on 3 November, so keep the two sets of outputs apart.

Two tools giving the same JA4 on one capture is a check that things work, not proof that every capture works.
