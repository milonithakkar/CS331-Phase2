# CS331 Computer Networks, Project 2: Where TLS Fingerprinting Tools Stop Working

Team T013, instructor Dr. Sameer G. Kulkarni

## Team

- Miloni Thakkar, 24110204
- Suhani, 24110358
- Yash Goyal, 24110399
- Laxmidhar Panda, 24110185

## About the project

In Project 1 we wrote our own JA3/JA3S/JA4 fingerprinting tool. Project 2 asked for something different: do not build another tool, take the ones that are used in practice (Wireshark/TShark and FoxIO's ja4) and find where they stop giving a correct or useful answer.

A TLS fingerprint is computed from the ClientHello, the first message of a TLS connection, which is sent in plaintext. Different fingerprints read different things: JA3 and JA4 read the ClientHello, JA4T the TCP SYN, JA4X the server certificate and JA4H the HTTP request. We first listed which fields each one uses and which RFC defines them ([`docs/landscape.md`](docs/landscape.md)). Then we changed one thing at a time and captured real traffic: QUIC instead of TCP, no TLS keys, the capture point and segmentation offload, a post-quantum key share, many connections from one browser, IPv6, and ECH. These are the seven tests L1 to L7 (Track 1), each written up in [`docs/tests/`](docs/tests/README.md).

We also wrote our own QUIC Initial extractor in C++ and checked its output byte for byte against TShark (Track 2). While testing it we found that the client's later Initial packets must be decrypted with keys from its first connection ID, even after the server has switched to its own ID, so we added a stateful mode for this.

## Results

### Track 1: seven tests with TShark and FoxIO ja4

| | What we changed | What we saw | Status |
|---|---|---|---|
| L1 | QUIC instead of TCP | TShark and FoxIO found 5/5 ClientHellos for curl and for Chromium, including Chromium's hello split over 21 CRYPTO frames | The tools handle it; our hypothesis was wrong |
| L2 | No TLS keys | JA4X + JA4H: 0 without keys, 5 with keys | Confirmed |
| L3 | Capture point and offload | Offload on: the ClientHello shows as one 1586-byte frame; off: 1514 + 138 bytes. Inside Docker the SYN-ACK (MSS 65495) comes from Docker's network stack, not from Cloudflare. On Windows the container's connection reaches the Wi-Fi card with Windows TCP values (TTL 128, window scale 8) | Confirmed in our setup |
| L4 | Post-quantum key share | ClientHello 508 bytes with X25519, 1511 bytes with X25519MLKEM768; JA4 changes (14 vs 12 extensions) | Confirmed |
| L5 | Same browser, many connections | 50 Chromium ClientHellos give 50 different JA3s. JA4: 15 values with TShark 4.6.8 and FoxIO v0.18.6, 1 value with FoxIO v1.0.1 | Confirmed: JA4 depends on the tool version |
| L6 | IPv6 | No IPv6 route on our networks, so the capture is empty | Not measured |
| L7 | ECH | The SNI shows `cloudflare-ech.com`, but plain DNS still asks for `crypto.cloudflare.com`. The DoH run did not load on the campus network | Confirmed; DoH not measured |

The numbers and commands for each test are in [`LEDGER.md`](LEDGER.md) and [`docs/tests/`](docs/tests/README.md).

### Track 2: our C++ QUIC Initial extractor

When the server answers, the client sends its next Initial packets to the server's new connection ID (RFC 9000 §7.2), but the keys for those packets still come from the first connection ID (RFC 9001 §5.2). An extractor that derives keys from each packet's own ID cannot decrypt them. With `--stateful-initial-context` our extractor remembers the first ID for each UDP 4-tuple.

| | curl | Chromium |
|---|---|---|
| client Initial packets | 20 | 16 |
| decrypted, keys from each packet's own ID (stateless) | 10 | 10 |
| decrypted, keys from the first ID (stateful) | 20 | 16 |
| complete ClientHellos, both modes | 5/5, same bytes as TShark 4.6.4 | 5/5, same bytes as TShark 4.6.4 |

TShark already decrypts these packets, and the stateful mode makes our extractor match it. In our captures the later packets only carry ACK and PADDING, so the ClientHello and JA4 are the same in both modes.

We also ran both modes on damaged copies of the two captures (reversed order, shuffled order, one packet dropped, every third CRYPTO packet dropped, one packet truncated) and compared with TShark on the same damaged file. The stateless mode matched TShark on all 54 ClientHellos. The stateful mode matched 42: it gets 0/5 on reversed order and 4/5 on shuffled order, because it takes the first packet it sees on a 4-tuple as the key source. More in [`docs/quic/implementation-status.md`](docs/quic/implementation-status.md).

## How to run

```bash
make test     # build the C++ code, run CTest and pytest
make check    # check every pcap against its SHA-256 in captures/manifest.json
python3 code/scripts/verify_all_seven_rows.py   # offline check of L1 to L7 (L6 is reported as not measured)
make lab-image && make lab-eval                 # Docker lab: rerun TShark and FoxIO on all pcaps
```

Running the C++ extractor on one capture:

```bash
./build/code/quic/src/quic-fingerprint --pcap captures/pcaps/quic_curl.pcap
./build/code/quic/src/quic-fingerprint --pcap captures/pcaps/quic_curl.pcap --stateful-initial-context
```

You need CMake, a C++20 compiler and the OpenSSL 3 headers (`libssl-dev`). The Python tests need `pytest` (`requirements.txt`). The Docker targets need Docker.

## Folder structure

The folders asked for in the submission:

```
README.md
code/      C++ QUIC extractor (code/quic) and Python capture, analysis and test scripts (code/scripts)
report/    final report (PDF) and its source
ppt/       presentation (.pptx) and its slide text
ai/        AI prompts and conversations
```

These were not required. We added them because the tests depend on them, or because we used them while working:

```
captures/        the 16 pcaps used in the tests, with SHA-256 hashes in manifest.json
results/         tool outputs, the 6 Nov re-check of Track 1, and the Track 2 test runs
docs/tests/      one write-up per test (T0 to T7)
docs/            fingerprint field notes (landscape.md, basics/), glossary, reading guide
docs/planning/   our implementation plan, proposal and design decisions, written while we worked
LEDGER.md        one line per test with the main number
CMakeLists.txt, Makefile, pyproject.toml, requirements.txt, .github/   build, tests and CI
```

## What we did not cover

- Track 1 was captured inside Docker on one Mac, on the campus network, which has no IPv6 and blocked DoH.
- One site (Cloudflare) and three clients (Chromium, curl, Python). TLS 1.2 was not tested.
- QUIC Retry, connection migration and QUIC v2. The stateful mode depends on packet order.

## AI use

All AI prompts and conversations are in [`ai/`](ai/).
