# Results

| Path | What it is |
|---|---|
| `summary.md`, `summary.json` | Track 1 results table, one row per capture. Our checked copy of Yash's summary; the only change is that the empty IPv6 row is marked not comparable. |
| `imported/` | Yash's original summary, unchanged. |
| `raw/<capture-id>.json` | The raw TShark and FoxIO output for each of the 15 Track 1 pcaps. |
| `track1/` | The capture log, the ECH page outputs, and the 6 November re-check of Track 1 numbers. |
| `track2/` | The three WSL verification runs, the QUIC connection ID inventory, and the earlier L3 probe note. |
| `l3_paired_batch/` | The Windows run for T3: the same requests captured inside the container, on WSL, on the vEthernet adapter and on the Wi-Fi card. |
| `baseline/` | TShark and FoxIO output for the early QUIC baseline capture. |
| `versions.txt` | Tool versions in Yash's Docker lab. |
| `preflight.json` | Tool versions on the WSL machine on 3 November. |
| `checkpoint_build_test.txt` | An early build and test log from Yash's lab. |
| `evaluation.json` | Output of `make eval`: the status of every capture in the manifest. |
| `quic-extractor-comparison.json` | The first comparison of the C++ extractor with TShark, from 3 November. The later WSL runs in `track2/` replace it. |
| `verification-status.md` | A summary of what the WSL runs and the Track 1 data show. |

`reproduced/` is ignored by git. `make lab-eval`, the scripts and the seven-row checker write fresh output there, so the committed results are never overwritten. Results should only change by rerunning the tools, never by editing numbers by hand.
