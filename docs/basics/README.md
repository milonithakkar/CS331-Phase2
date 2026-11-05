# Fingerprint field cards

One short card per fingerprint: which message it reads, which fields go in, how they are combined, and what you can see without TLS keys. The comparison of all seven is in [`../landscape.md`](../landscape.md).

| Card | Reads | Defined by |
|---|---|---|
| [JA3](ja3.md) | client ClientHello | Salesforce JA3 repository |
| [JA3S](ja3s.md) | server ServerHello | Salesforce JA3 repository |
| [JA4](ja4.md) | client ClientHello (TCP or QUIC) | FoxIO JA4+ v1.0.1 |
| [JA4S](ja4s.md) | server ServerHello (TCP or QUIC) | FoxIO JA4+ v1.0.1 |
| [JA4T](ja4t.md) | client TCP SYN | FoxIO JA4+ v1.0.1 |
| [JA4X](ja4x.md) | X.509 certificate | FoxIO JA4+ v1.0.1 |
| [JA4H](ja4h.md) | HTTP request | FoxIO JA4+ v1.0.1 |

None of these is an IETF standard. The RFCs linked in each card define the protocol fields; the fingerprint recipe comes from Salesforce or FoxIO. Sources were checked on 5 November 2026.
