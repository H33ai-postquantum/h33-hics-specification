# HICS Scoring Specification v1.0

**H33 Independent Code Scoring**

Algorithm Version: 1.0.0
Effective: March 30, 2026
Published by: H33.ai, Inc.

---

## 1. Overview

HICS (H33 Independent Code Scoring) is an open-source scoring standard for evaluating software codebases across five weighted dimensions. The scoring formula, category definitions, weights, and grade thresholds are published here for public audit.

The implementation (AST scanners, STARK proof generation, Dilithium signing infrastructure) is proprietary. The methodology is transparent. The technology is licensed.

The algorithm is the authority. H33 does not editorialize scores.

## 2. Composite Score

```
Final = (Crypto × 0.30) + (Vuln × 0.25) + (Data × 0.20) + (Ops × 0.15) + (Health × 0.10)
```

Each category scored 0–100. Final score rounded to nearest integer.

### Grade Thresholds

| Grade | Range |
|-------|-------|
| A | 90–100 |
| B | 80–89 |
| C | 70–79 |
| D | 60–69 |
| F | 50–59 |
| F- | 0–49 |

## 3. Categories

### 3.1 Cryptographic Security (30%)

Detects PQ-vulnerable algorithms, classical misuse, key management failures.

**Finding Types:**
- PQ-Vulnerable: ECDH, ECDSA, RSA, Ed25519, secp256k1/r1, DH classical
- Classical Misuse: MD5, SHA-1, DES, RC4, AES-ECB, AES-CBC (no AEAD), Blowfish
- Key Management: Hardcoded secrets (Shannon entropy-weighted)
- JWT: RS256, ES256, PS256 (classical signing)

**Positive Credits (capped at +15):**
- Kyber/ML-KEM: +4.0 per detection
- Dilithium/ML-DSA: +4.0 per detection
- FALCON: +3.0 per detection
- SPHINCS+/SLH-DSA: +2.0 per detection

**Hybrid Detection:** If a file imports both Ed25519 AND Dilithium, the Ed25519 finding is skipped. This recognizes NIST SP 1800-38 hybrid migration as defense-in-depth.

**Files Excluded:** pqc/, fhe/, zkp/, tee/, mpc/, biometric_auth/ (crypto library implementations)

### 3.2 Vulnerability Surface (25%)

AST-based detection of injection, auth bypass, XSS, SSRF, credentials.

**Finding Types:**
- SQL Injection: raw_sql/rawQuery with string interpolation
- Command Injection: subprocess/os.system/child_process with variable input
- Deserialization: pickle, yaml.load (without SafeLoader), unserialize
- Hardcoded Credentials: Variable named password/secret/token assigned string literal
- JWT None: Algorithm set to "none" (HARD FAIL)
- XSS: dangerouslySetInnerHTML, v-html
- CSRF: Protection explicitly disabled

**Credential Skip Rules:**
- Values containing: todo, xxx, changeme, test_, _test, mock, dummy, fake, sample, example, placeholder
- Values starting with: whsec_, sk_test_, pk_test_, rk_test_
- Values in test code under 30 characters
- Env references ($env, process., os.environ, getenv)

**Files Excluded:** Cargo.toml, package.json, go.mod (dependency manifests)

### 3.3 Data Handling & Privacy (20%)

PII exposure, encryption at rest, GDPR/HIPAA compliance patterns.

**Finding Types:**
- PII in Logs: password/secret/ssn/credit_card in log call arguments
  - "secret" only flagged when adjacent to interpolated value (secret: {})
  - Skipped if log text contains: "not configured", "not set", "missing", "required"
- Plaintext Password Storage: password = value (with storage context)
- SSN/Credit Card: Luhn-validated or formatted numbers in assignments
- No Encryption at Rest: Flat deduction if no AES/KMS/FHE across ALL files
- No GDPR Consent: Flat deduction if no consent mechanism detected
- No Right to Erasure: Flat deduction if no deletion endpoint detected

**Indicator Detection:** Runs on ALL files (including crypto libraries) for encryption-at-rest, consent, and erasure checks. Per-file vulnerability scanning skips crypto libraries.

### 3.4 Operational Resilience (15%)

Error handling, external service resilience, rate limiting, observability.

**Finding Types:**
- Panic Points: Bare .unwrap() (not .expect()) in non-test, non-startup code, grouped by file
- HTTP Error Handling: External API calls without catch/try/map_err/?/Result
- HTTP Timeouts: External calls without .timeout() in file
- N+1 Queries: ORM-specific patterns (.findOne/.findById) inside JS/Python loops
- Flat: No rate limiting, no auth rate limiting, no health check, no structured logging

**Startup Exclusions for .unwrap():** env, config, init, new(), parse, bind, listen, build(), builder, default(), connect, setup, create, open, compile, regex, from_str, try_from, addr, socket, channel, mutex, lock, read_dir, metadata, canonicalize

**Client Construction Skip:** reqwest::Client::builder() and ::new() are not flagged as "unhandled HTTP calls" — they're client construction, not API calls.

**Error Handling Recognition:** .catch, try, except, .map_err, ?;, match, if let Err/Ok, .unwrap_or, .unwrap_or_default, .unwrap_or_else, .and_then, .ok(), .expect(), Result<, -> Result

### 3.5 Code Health & Maintainability (10%)

Test coverage, CI/CD, complexity, project hygiene. Advisory findings.

**Finding Types:**
- Test Ratio: test_lines / code_lines (inline #[cfg(test)] counted)
  - < 0.1: -5.0 (low)
  - < 0.3: -3.0 (moderate)
- Complexity: Nesting depth > 10, function length > 500 lines, file length > 2000 lines
- No CI/CD: -3.0 (filesystem check for .gitlab-ci.yml, .github/workflows, etc.)
- Security TODO: FIXME/TODO referencing vulnerability/injection/xss/csrf/insecure/unsafe/cve/exploit
- No README: -2.0 | No LICENSE: -2.0 | No CHANGELOG: -0.5
- Excessive TODOs (>50): -1.0 | Excessive deps (>100): -1.0

**Complexity Skip Modules:** zk_verify, zk_phish, botshield, fraud_shield, infra, middleware, solana, cache, rpc_client, blockchain, ai_compliance, compliance, policy_engine, audit, parser, handler, proof, scanner, vaultkey, secret_store, fednow, xml, protocol, codec, serializ, deserializ, quantumvault, archival, proxy, server.rs, zk_proven, ml_telemetry, medvault, phi_store, token_distribution, reports, .html

**README/LICENSE/CHANGELOG Detection:** Filesystem-level check (not in scanned code files). Engine injects synthetic markers (__readme_detected__, etc.).

## 4. Confidence Weighting

```
deduction = base_deduction × confidence
```

Confidence range: 0.0–1.0. Shannon entropy for secret detection:
- Entropy > 4.5: confidence 0.95
- Entropy > 4.0: confidence 0.85
- Entropy > 3.5: confidence 0.70
- Entropy > 3.0: confidence 0.50
- Entropy ≤ 3.0: confidence 0.30

## 5. Density Caps

Maximum total deduction per finding type. Prevents single-pattern dominance.

| Type | Cap |
|------|-----|
| health.large_file | 3.0 |
| health.long_function | 4.0 |
| health.high_complexity | 5.0 |
| health.security_todo | 4.0 |
| ops.no_error_handling | 9.0 |
| ops.no_timeout | 12.0 |
| ops.panic_on_input | 8.0 |
| vuln.sql_injection | 24.0 |
| vuln.command_injection | 18.0 |
| **No cap:** hardcoded, jwt_none, ssn, credit_card, cvv, plaintext_password | Unlimited |
| Default (unlisted types) | 20.0 |

## 6. Test Code Multiplier

Findings in test code: `deduction × 0.25`

## 7. Hard Fails

Force category to 0/100:
- JWT "none" algorithm in production code
- Hardcoded production credential (high entropy, non-test)

## 8. Attestation

- **Merkle Root:** SHA3-256 tree over all scanned files
- **STARK Proof:** Computational integrity. No trusted setup. Hash-based (quantum-resistant).
- **Dilithium ML-DSA-65:** NIST FIPS 204. Post-quantum signature.
- **Proof ID:** SHA3-256(certificate). Permanent. Verifiable at h33.ai/verify.

## 9. Claims Verification (HICS-V)

21 known claim types across code-detectable (Category A) and compliance (Category D) categories. Vendor submits claims.yaml. Engine evaluates each claim as Verified / Divergent / Unverified.

Undisclosed strengths (detected but not claimed) and weaknesses (expected but absent) are surfaced automatically.

## 10. Change Policy

H33 may update this specification at any time. Substantive changes increment the version number. Historical scores reflect the algorithm version active at scan time. Scores are not retroactively updated.

---

Published by H33.ai, Inc. | Florida, United States
Terms: h33.ai/hics/terms | Methodology: h33.ai/hics/methodology
The algorithm is the authority.
