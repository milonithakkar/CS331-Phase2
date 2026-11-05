# Glossary

Terms used in this repository, in plain words. Each section builds on the one before it.

## 1. Standards

| Term | Meaning |
|---|---|
| IETF | Internet Engineering Task Force, the group that writes internet protocol standards. |
| RFC | "Request for Comments", an IETF document that defines a protocol. RFC 8446 is TLS 1.3; "RFC 8446 §4.1.2" means section 4.1.2 of it. |
| Draft | An IETF document that is not an RFC yet and can still change. |
| IANA | Keeps the official lists of protocol numbers, for example which number means which cipher or group. |

## 2. Network basics

| Term | Meaning |
|---|---|
| Packet | One piece of data sent over the network. Large messages are split into many packets. |
| Frame | A packet as seen on the link, including the Ethernet header. tcpdump records frames. |
| IPv4 / IPv6 | The addressing layer. IPv4 looks like `104.16.124.96`; IPv6 is the newer, longer format. |
| Privacy addresses | IPv6 addresses that change over time so a device is harder to track by address (RFC 8981). |
| TCP | Reliable connection on top of IP, used by normal HTTPS. Starts with SYN, SYN-ACK, ACK. |
| SYN / SYN-ACK | The first two packets of a TCP connection. They carry TCP options such as MSS and window scale. |
| ISN | Initial sequence number, picked at random in each SYN. Two SYNs with different ISNs belong to different TCP connections. |
| UDP | Simple transport with no connection. QUIC runs on top of it. |
| Port | Number that says which service a packet is for. HTTPS is 443, DNS is 53. |
| MTU | Largest packet a link allows. Usually 1500 bytes. |
| MSS | Largest TCP payload a side wants to receive, sent in the SYN. Usually MTU minus 40, so 1460. |
| TTL | Counter in every IP packet, reduced by 1 at each router. Linux starts at 64, Windows at 128. |
| Segment | One TCP packet's worth of data. A large ClientHello can need two. |
| TSO / GSO / GRO | Offload: the kernel hands one big packet to the network card to cut up later (TSO/GSO, sending), or joins many packets into one (GRO, receiving). A capture taken before that point shows packets that never existed on the wire. |
| NAT | Rewrites addresses, like a home router. The TCP connection itself passes through. |
| Proxy | Ends your connection and opens a new one on your behalf. |
| Capture point / tap | Where the capture is taken. A tap only listens. |
| DNS | Turns a name like `crypto.cloudflare.com` into an IP address. Plaintext on port 53 by default. |
| DoH | DNS over HTTPS: the lookup is sent inside HTTPS so others cannot read it (RFC 8484). |
| HTTPS record | A DNS record type (RFC 9460). Among other things it carries the server's ECH key. |

## 3. TLS

| Term | Meaning |
|---|---|
| TLS | Transport Layer Security, the encryption in HTTPS. TLS 1.3 is RFC 8446. SSL is its old name. |
| Handshake | The first messages, where client and server agree on how to encrypt. |
| ClientHello | The client's first handshake message. Over TCP it is plaintext, unless ECH hides the inner one. In QUIC it is inside Initial packets, whose keys anyone can compute. This is what JA3 and JA4 read. |
| ServerHello | The server's reply: picks one cipher and key share. Plaintext over TCP; in QUIC it is in the server's Initial packets. |
| Cipher suite | A named set of encryption algorithms, e.g. `0x1302` = AES-256-GCM with SHA-384. |
| Extension | An optional field in the hello, each with a number, e.g. `0x0000` = SNI, `0x0010` = ALPN. |
| SNI | Server Name Indication: the site name in the ClientHello. With ECH the real name is in the encrypted inner hello. |
| ALPN | Which application protocol runs inside TLS: `h2` (HTTP/2), `http/1.1`, `h3` (HTTP/3). |
| Signature algorithms | Extension listing which signature types the client accepts (RFC 8446 §4.2.3). |
| Supported groups | Extension listing which key exchange methods the client supports. |
| Key share | The client's public key for agreeing on a secret (RFC 8446 §4.2.8). |
| X25519 | A common classic key exchange group, number 29. |
| Post-quantum (PQ) | Crypto meant to stay safe against future quantum computers. |
| ML-KEM | A post-quantum key exchange method standardised by NIST. Its keys are about 1.2 KB. |
| X25519MLKEM768 | A hybrid group, X25519 and ML-KEM together. Number 4588 (0x11EC). |
| GREASE | Reserved fake values (`0x0a0a`, `0x1a1a` … `0xfafa`) that clients insert so servers learn to ignore unknown values (RFC 8701). Fingerprints are supposed to skip them. |
| Padding extension | Filler bytes that change the hello's size (RFC 7685). OpenSSL adds it to hellos of 256 to 511 bytes when the app turns the option on, to avoid a bug in old F5 load balancers. curl's small hello in T4 had it. |
| ec_point_formats | An old extension about elliptic curve point formats (RFC 8422). |
| Certificate / X.509 | The server's signed identity document. Encrypted in TLS 1.3. |
| Certificate compression / abridged certificates | Ways to send a smaller certificate (RFC 8879) or only a reference to it (newer drafts). |
| Key log / SSLKEYLOGFILE | If set, curl and Chrome write their session keys to a file so Wireshark can decrypt that session. A real network tap never has it. |
| AEAD | The kind of encryption TLS 1.3 and QUIC use, which encrypts and checks for tampering at once (e.g. AES-GCM). |
| ECH | Encrypted ClientHello: the real ClientHello is encrypted inside an outer one that shows only a shared name (RFC 9849). |
| eSNI | An older draft that encrypted only the SNI; replaced by ECH. |
| public_name | The shared name in the outer ClientHello when ECH is used. We saw `cloudflare-ech.com`. |

## 4. QUIC

| Term | Meaning |
|---|---|
| QUIC | Transport over UDP with TLS 1.3 built in (RFC 9000, RFC 9001). Used by HTTP/3. |
| HTTP/3 | HTTP over QUIC. |
| Initial packet | The first QUIC packets in each direction. The client's carry the ClientHello. They are encrypted, but with keys anyone can compute. |
| Connection ID | An ID in each QUIC packet. The Initial keys are made from the destination connection ID of the client's first Initial. |
| Connection ID switch | After the server replies, the client sends its next Initials to the server's chosen ID, but the keys still come from the first ID (RFC 9000 §7.2, RFC 9001 §5.2). |
| Salt | A fixed public value mixed into key derivation. QUIC v1's Initial salt is in RFC 9001 §5.2. |
| Header protection | A mask over the packet number in QUIC packets (RFC 9001 §5.4). |
| CRYPTO frame | A piece of the TLS handshake inside a QUIC packet. A ClientHello can be split over many; Chromium's was in 21. |
| Connection migration | A QUIC connection moving to a new network path without a new handshake (RFC 9000 §9). |
| Retry / 0-RTT / QUIC v2 | Other QUIC features and versions we did not test. |

## 5. Fingerprints

| Term | Meaning |
|---|---|
| Fingerprint | A short string made from how a client fills its hello. It points to the software (library and settings), not the person. |
| Hash | Turns any input into a short fixed-length string. Same input, same output. |
| MD5 / SHA-256 | Two hash functions. JA3 uses MD5 (32 hex characters). JA4 uses SHA-256 cut to 12 hex characters. |
| JA3 | Salesforce's fingerprint: MD5 of version, ciphers, extensions, groups and point formats, in the order sent. |
| JA3S | JA3 for the ServerHello. |
| JA4 | FoxIO's fingerprint, `a_b_c`: a is a readable summary, b a hash of the sorted ciphers, c a hash of the sorted extensions and signature algorithms. |
| ja4_r | The raw JA4: the same parts before hashing, so you can see the values. |
| JA4S | JA4 for the ServerHello. |
| JA4X | Fingerprint of a certificate. |
| JA4H | Fingerprint of an HTTP request. |
| JA4T / JA4TS | Fingerprint of the TCP SYN / SYN-ACK (window, options, MSS, window scale). |
| JA4L | Latency between client and server. |
| JA4+ | The whole FoxIO family above. |
| Collision | Two different apps giving the same fingerprint. |
| Stable | The same app always gives the same fingerprint. |

## 6. Tools

| Term | Meaning |
|---|---|
| pcap | A file of captured packets. Ours are in `captures/pcaps/`. |
| tcpdump | Command-line program that captures packets into a pcap. |
| Wireshark / TShark | The most used packet analyser, written in C. TShark is its command-line version. It computes JA3 and JA4 too. |
| FoxIO ja4 | FoxIO's JA4+ command-line tool, written in Rust. It reads packets through TShark. |
| ja4-main | Our build of FoxIO's unreleased code at commit c6469a7, used to compare with the release. |
| Release / tag / commit | A commit is one saved change in git; a tag names a commit as a release, e.g. `v0.18.6`. |
| Chromium | The open-source browser Chrome is built from. We used version 156. |
| curl | Command-line program that downloads URLs. Uses OpenSSL here. |
| OpenSSL | The TLS library used by curl and Python on Linux. |
| Docker / container / image | Docker runs a small Linux system (a container) from a recipe (an image, built from `code/Dockerfile`). Our Track 1 lab runs in it. |
| make | Runs short named commands like `make lab-eval` from the Makefile. |
| Manifest | `captures/manifest.json`: the list of all captures with their SHA-256 hashes. |
| pytest | Python test runner. |
| CMake / CTest | Build and test tools for the C++ code. |
| Sanitizers (ASan/UBSan) | Compiler options that stop the program loudly on memory bugs. |

## 7. Our method

| Term | Meaning |
|---|---|
| Hypothesis | What we expect before testing. It can turn out wrong (T1 did). |
| Control | The normal case we compare against. |
| Variable | The one thing we change in a test. |
| Limitation (L1 to L7) | A situation where the real tools do not give a useful or correct answer. |
| Track 1 | The seven tests with TShark and FoxIO on real captures. |
| Track 2 | Our C++ QUIC Initial extractor and its comparison with TShark. |
