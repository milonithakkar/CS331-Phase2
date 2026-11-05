# Setting up on Ubuntu / WSL and Docker

There are two lab setups in this project and their tool versions are kept apart:

- the WSL machine used for Track 2: TShark 4.6.4, ja4 binary reporting 0.18.6 (`results/preflight.json`),
- Yash's Docker lab used for Track 1: TShark 4.4.19, FoxIO tag v0.18.6 (reports 0.18.5) and FoxIO main as `ja4-main` (`results/versions.txt`).

## Build and test

```sh
sudo apt update
sudo apt install -y build-essential cmake libssl-dev python3 python3-venv make
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt

make test     # build the C++ code, run CTest and pytest
make check    # check the pcap hashes
make eval     # write a status summary of the captures to results/evaluation.json
```

The Makefile uses `.venv/bin/python` automatically if it exists. `make preflight` writes the current machine's tool versions to `results/preflight-current.json`. TShark and ja4 are not needed for the tests, only for rerunning the reference tools.

## Run the QUIC extractor

```sh
make build
build/code/quic/src/quic-fingerprint --pcap captures/pcaps/quic_chromium.pcap
build/code/quic/src/quic-fingerprint --pcap captures/pcaps/quic_chromium.pcap --stateful-initial-context
```

To get TShark's ClientHellos from the same file for comparison:

```sh
sudo apt install -y tshark
tshark -r captures/pcaps/quic_chromium.pcap -Y 'tls.handshake.type == 1' -T jsonraw > /tmp/quic_chromium-clienthellos.json
```

Compare the full ClientHello bytes, not only the lengths or the JA4 string.

## Damaged copies

```sh
python3 code/scripts/quic_impairment.py \
  --input captures/pcaps/quic_chromium.pcap \
  --work-dir results/reproduced/quic_impairment/chromium \
  --extractor build/code/quic/src/quic-fingerprint --tshark tshark --seed 331
```

Do the same with `quic_curl.pcap`, and again with `--stateful-initial-context` in a separate work directory. Each run makes five damaged copies (reversed order, shuffled order, one CRYPTO packet dropped, every third CRYPTO packet dropped, one CRYPTO packet cut by 64 bytes), runs the extractor and TShark on each, and writes the comparison to `impairment-results.json`. `cases.json` lists which frames were changed. The committed pcaps are never modified.

For a damaged copy where the ClientHello is incomplete, the extractor prints the expected length (if the 4-byte header survived), how many bytes it saw, and which byte ranges are missing. To see which CRYPTO frames were in the original:

```sh
tshark -r captures/pcaps/quic_curl.pcap -Y 'quic.crypto.length > 0' -T fields \
  -e frame.number -e quic.long.packet_type -e quic.dcid -e quic.crypto.offset -e quic.crypto.length
```

## Track 1 Docker lab

```sh
make lab-image    # needs network for the build
make lab-eval     # reads the committed pcaps, writes to results/reproduced/
```

The image installs TShark, tcpdump, Chromium, curl, FoxIO ja4 at tag v0.18.6 and FoxIO main as `ja4-main`. `make lab-eval` does not create any traffic and does not overwrite the committed results.

To make a new capture:

```sh
make lab-capture CAPTURE_ID=quic_chromium
```

This needs network access and writes to `captures/reproduced/`. It is not added to the manifest automatically. Check it before adding it, and keep TLS key logs out of the repository.
