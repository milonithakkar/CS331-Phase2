# T0: Build, tests and capture checks

Before trusting any number, we check that the code builds, the tests pass, and the committed pcaps are the ones the results were computed from.

## What is checked

- The C++ QUIC extractor builds, and its test program passes (CTest).
- The Python tests pass (pytest). They cover the capture manifest and hashes, the reference-tool runner, the Track 1 scripts, the QUIC damage harness and the seven-row checker.
- Every pcap matches the SHA-256 recorded in `captures/manifest.json`.
- Tool versions for the Docker lab are in `results/versions.txt`.

## Commands

```sh
make test     # build, CTest, pytest
make check    # pcap hashes against the manifest
```

GitHub Actions runs both on every push (`.github/workflows/ci.yml`).

## Result

The last recorded run, in `results/track2/verification-batch-20261105-105416/make-test.log`, has CTest 1/1 and pytest 52 passed. The source files that batch recorded hashes for are unchanged in the repository. `make check` reports 16 captures: 15 with packets and the empty IPv6 one.

`results/checkpoint_build_test.txt` is an older log from Yash's lab (CTest 1/1, 9 pytest tests, 1/1 with sanitizers) from before most of the tests existed.

## What it does not show

Passing tests show the code does what the tests check. They do not show that the C++ extractor is right on traffic it has not seen. That is what the comparison with TShark in T1 is for.
