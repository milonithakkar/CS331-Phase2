# Scripts

Python runs the captures, the reference tools and the checks; the QUIC parsing and crypto are in the C++ code under `code/quic/`. The scripts save the exact commands and raw tool outputs, so every number can be traced back to a run.

| Script | What it does |
|---|---|
| `capture.py` | Makes one Track 1 capture again from its recipe. Only runs with `--confirm-network` and writes to `captures/reproduced/`. Used by `make lab-capture`. |
| `analyze.py` | Runs TShark and FoxIO on all Track 1 pcaps and writes a fresh summary to `results/reproduced/`. Used by `make lab-eval`. |
| `versions.sh` | Records the lab's tool versions (`make lab-versions`). |
| `preflight.py` | Records CMake, compiler, OpenSSL, TShark and FoxIO versions and the TShark fields available. `make preflight` writes `results/preflight-current.json`. |
| `reference_baseline.py` | Runs TShark and FoxIO on one registered capture and saves the outputs to `results/reference/` (`make baseline`). |
| `evaluate.py` | Checks the manifest, hashes, metadata and packet counts (`make check`, `make eval`). |
| `quic_initial_inventory.py` | Lists the QUIC v1 Initial headers in a pcap: UDP endpoints and connection IDs, without decrypting. |
| `quic_impairment.py` | Makes damaged copies of a QUIC pcap (reordered, dropped or cut packets) and compares the C++ extractor's ClientHellos with TShark's on each copy. |
| `l3_proxy_sanity.py` | Reads the SYN and SYN-ACK values per TCP stream from one or more pcaps, and compares streams that were paired by hand. |
| `verify_all_seven_rows.py` | Checks the committed results for L1 to L7; `--live` also reruns the tools. |
| `capture_ipv6_egress.sh` | Captures a `curl -6` and `curl -4` request to the same host on a network with IPv6. |
| `verify_submission_batch.sh` | Runs the build, tests and all Track 2 checks in one go into a new folder under `results/track2/`. |

The tests for these scripts are in `tests/` and run with `make test`. More on running them in [`docs/ubuntu-setup.md`](../../docs/ubuntu-setup.md) and [`docs/verification-batch.md`](../../docs/verification-batch.md).
