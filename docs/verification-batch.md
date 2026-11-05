# Verification batches

`code/scripts/verify_submission_batch.sh` runs all the repeatable Track 2 checks in one go and saves everything in a new dated folder under `results/track2/`. It does not commit anything.

## What one batch does

1. Records the commit, branch and tool versions (`environment.txt`) and the SHA-256 of the input pcaps and of the scripts and C++ files it runs.
2. Runs `make check` and `make test`.
3. Makes the header-only connection ID inventory of the two QUIC pcaps.
4. Reads the SYN and SYN-ACK values from the two offload pcaps.
5. Runs the C++ extractor and TShark on the clean and damaged copies of both QUIC pcaps, in stateless and in stateful mode, and compares the ClientHello bytes. Seed 331.
6. Runs the seven-row checker (below).

## The three batches in the repository

| folder | what it adds |
|---|---|
| `verification-batch-20261104-201936` | first run, stateless mode only: 54/54 ClientHellos equal to TShark |
| `verification-batch-20261105-054047` | the same, run again on 5 November, same result |
| `verification-batch-20261105-105416` | adds the stateful mode and the seven-row checker; this is the one the README and report use |

Results of the 5 November batch (TShark 4.6.4, GCC 15.2.0, OpenSSL 3.5.5, Python 3.14.4, WSL2 kernel 6.18.33.2):

- `make test`: CTest 1/1, pytest 52 passed. `make check`: 16 captures, 15 with packets and the empty IPv6 one.
- Stateful mode on the captured order: all 20 curl and 16 Chromium client Initials decrypt, and all 5 ClientHellos per capture equal TShark's. Stateless mode decrypts 10 of 20 and 10 of 16.
- Damaged copies: stateless 27/27 per capture; stateful 21/27 per capture, because reversed order gives 0/5 and shuffled order 4/5. Details in [`quic/implementation-status.md`](quic/implementation-status.md).
- Offload SYNs: in both offload pcaps the client SYN is MSS 1460 / window scale 10 / window 64240 / TTL 64 and the SYN-ACK is 65495 / 7 / 65408 / 63. Why the SYN-ACK looks like that is explained in [`tests/T3_vantage_offload.md`](tests/T3_vantage_offload.md).
- Seven-row checker: L1, L2, L3, L4, L5 and L7 pass, L6 is not measured. The optional IPv6 loopback capture failed (tcpdump got no packets), so the "live" part reports FAIL.

The batch was run on the arena branch at commit `fa9fcdb`, before it was merged into main. The source files it hashed are unchanged on main.

## Running a new batch

From the repository root on Linux or WSL, with CMake, a C++20 compiler, the OpenSSL headers and TShark installed:

```sh
git pull
code/scripts/verify_submission_batch.sh
```

It refuses to overwrite an existing folder, so every run gets its own. Check the output for private data and size before committing it.

## The seven-row checker

```sh
python3 code/scripts/verify_all_seven_rows.py                 # check the committed results for L1 to L7
python3 code/scripts/verify_all_seven_rows.py --live          # also rebuild the C++ code and rerun TShark on the committed pcaps
python3 code/scripts/verify_all_seven_rows.py --ipv6-loopback # also try a TLS capture over ::1
```

The summary goes to `results/reproduced/seven_rows_verification/summary.json` (change it with `--output`). Each row is PASS, FAIL, BLOCKED (a tool is missing), NOT_RUN or NOT_MEASURED. A PASS means the committed results are consistent; it is not a new measurement. The checker reads the Track 2 numbers from the first batch folder (`20261104-201936`), so it does not check the stateful numbers.

The `::1` capture only shows that our IPv6 capture path works on the local machine. It says nothing about public IPv6, so L6 stays NOT_MEASURED.

## Measuring IPv6 on a network that has it

```sh
sudo code/scripts/capture_ipv6_egress.sh "results/track2/l6_ipv6_$(date -u +%Y%m%dT%H%M%SZ)" example.com
```

This records the route, interface and tool versions, runs `curl -6` and `curl -4` to the same host, captures both, and extracts the SYN and ClientHello with TShark. With no global IPv6 route it reports NOT_MEASURED instead of passing. It needs a network with real IPv6, such as a phone hotspot that has it; WSL on the campus network did not.
