# HICS — H33 Independent Code Scoring

The open-source scoring standard for software quality evaluation.

## What is HICS?

HICS evaluates codebases across five weighted dimensions — Cryptographic Security, Vulnerability Surface, Data Handling, Operational Resilience, and Code Health — producing a score from 0 to 100. The scoring formula is published here for public audit.

The free CLI runs locally. No data leaves your machine. No account required.

## Specification

- **[HICS-SPEC-v1.0.md](HICS-SPEC-v1.0.md)** — Complete scoring methodology
- **[h33.ai/hics/methodology](https://h33.ai/hics/methodology)** — Web version

## Key Properties

- **Open formula:** Weights, thresholds, finding types, density caps — all published
- **Proprietary implementation:** AST scanners, STARK proofs, Dilithium signing
- **STARK attested:** Every score is provably correct (zero-knowledge, quantum-resistant)
- **Dilithium signed:** Post-quantum signature (NIST FIPS 204)
- **No trust required:** Verify any score at [h33.ai/verify](https://h33.ai/verify)

## Scoring Formula

```
Final = (Crypto × 0.30) + (Vuln × 0.25) + (Data × 0.20) + (Ops × 0.15) + (Health × 0.10)
```

## Run It

```bash
brew install h33/tap/hics && hics scan .
```

Free. Local. No account. No network calls.

## Governance

This specification is maintained by H33.ai, Inc. Algorithm changes are versioned and documented. Historical scores reflect the algorithm version active at scan time. The algorithm is the authority.

## Terms

[h33.ai/hics/terms](https://h33.ai/hics/terms) — Florida jurisdiction.

## License

The HICS specification (this document) is published under CC BY 4.0.
The HICS implementation is proprietary to H33.ai, Inc.

---

H33.ai, Inc. | [h33.ai](https://h33.ai) | The algorithm is the authority.

---

**H33 Products:** [H33-74](https://h33.ai) · [Auth1](https://auth1.ai) · [Chat101](https://chat101.ai) · [Cachee](https://cachee.ai) · [Z101](https://z101.ai) · [RevMine](https://revmine.ai) · [BotShield](https://h33.ai/botshield)

*Introducing H33-74. 74 bytes. Any computation. Post-quantum attested. Forever.*


---

## H33 Product Suite

| Product | Description |
|---------|-------------|
| [H33.ai](https://h33.ai) | Post-quantum security infrastructure |
| [V100.ai](https://v100.ai) | AI Video API — 20 Rust microservices, post-quantum encrypted |
| [Auth1.ai](https://auth1.ai) | Multi-tenant auth without Auth0 |
| [Chat101.ai](https://chat101.ai) | AI chat widget with Rust gateway sidecar |
| [Cachee.ai](https://cachee.ai) | Sub-microsecond PQ-attested cache |
| [Z101.ai](https://z101.ai) | 20+ SaaS products on one backend |
| [RevMine.ai](https://revmine.ai) | Revenue intelligence platform |
| [BotShield](https://h33.ai/botshield) | Free CAPTCHA replacement |

*Introducing H33-74. 74 bytes. Any computation. Post-quantum attested. Forever.*
