# HICS Specification Changelog

## v1.0.0 — March 30, 2026

Initial release.

- 5 scoring categories with weights (30/25/20/15/10)
- 21 code-detectable claim types (HICS-V)
- Confidence-weighted deductions with Shannon entropy
- Density caps per finding type
- Hybrid PQ scheme detection (defense-in-depth credit)
- Tree-sitter AST parsing for Rust, Python, JS, TS
- STARK attestation + Dilithium ML-DSA-65 signing
- Grade thresholds: A (90+), B (80+), C (70+), D (60+), F (50+), F- (<50)
- Test code 25% weight multiplier
- Hard fails: JWT "none", high-entropy hardcoded production credentials
- H33 self-assessment: 100/100 (Grade A)
