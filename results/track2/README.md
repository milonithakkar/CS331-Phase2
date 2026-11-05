# Track 2 results

## Verification runs

Each `verification-batch-*` folder is one run of `code/scripts/verify_submission_batch.sh` on the WSL machine. Each holds the environment and tool versions, the hashes of the input pcaps and scripts, the build and test logs, the QUIC connection ID inventory, the SYN analysis of the offload pcaps, and the damaged-copy comparison: for each case the damaged pcap, TShark's raw output, the extractor's log and a result JSON.

| folder | source commit | contents |
|---|---|---|
| `verification-batch-20261104-201936` | `619a694` | stateless mode only; 27/27 per capture, 54/54 in total |
| `verification-batch-20261105-054047` | `f642d1a` | the same checks again, same result |
| `verification-batch-20261105-105416` | `fa9fcdb` (arena branch) | adds the stateful mode and the seven-row checker; the README and report use this one |

Every comparison is against TShark on the same pcap. In the every-third-packet-dropped case TShark itself only finds 2 ClientHellos, so 2/2 there means we agree with TShark. The full results are explained in [`docs/quic/implementation-status.md`](../../docs/quic/implementation-status.md).

Do not edit a run folder. A new run goes into a new folder.

## Other files

- `quic-initial-dcid-inventory.json`: the QUIC Initial headers of the two QUIC pcaps (UDP endpoints, connection IDs, packet numbers), made by `code/scripts/quic_initial_inventory.py`. It shows, for each of the five flows per capture, the client's later Initials using the server's connection ID.
- `l3-paired-probe-c28a486.md`: notes on an earlier L3 probe, before the paired Windows run.
- `l3-live-paired-probe-20261105/`: pcaps and comparison from a paired L3 probe on 5 November. The full paired run used in T3 is in `results/l3_paired_batch/`.

## Running the damaged-copy comparison alone

```sh
python3 code/scripts/quic_impairment.py \
  --input captures/pcaps/quic_curl.pcap \
  --work-dir results/reproduced/quic_impairment/curl \
  --extractor build/code/quic/src/quic-fingerprint --tshark tshark --seed 331
```

Add `--stateful-initial-context` for the stateful mode. Output goes to the ignored `results/reproduced/`.
