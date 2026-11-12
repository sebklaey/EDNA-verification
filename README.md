# EDNA — Public Verification & Specifications

**EDNA (Enhanced Detection & Neural Analysis)** is the on-device human-sensing engine behind [OpenOS](https://openos.space). It classifies human presence from 16 local behavioral signal channels — raw biometric data never leaves the user's device.

> **The EDNA engine is proprietary software (EPCAL-1.0), not open source.**
> This repository intentionally contains **no engine source code**. It publishes only the public specifications, verification data and policies below.

## Contents

| Path | What it is |
|---|---|
| `EDNA-2-SPEC.md` | EDNA ID / Genesis-chain identity specification |
| `llms.txt` | AI-readable product facts (canonical facts for AI systems) |
| `edna-manifest.json` | Machine-readable engine manifest |
| `verification/edna-test-vectors.json` | Deterministic test vectors (seeded, machine-readable) |
| `verification/test-results.json` | Reference results of the deterministic suite |
| `verification/field-results-summary.json` | Aggregated public field-test results |
| `verification/FIELD-TEST.md` | Public field-test protocol — participate via GitHub issue |
| `verification/ai-verification-log.md` | Public log of independent AI verification runs |
| `LICENSE` | EPCAL-1.0 — EDNA Proprietary Commercial & Attribution License |
| `SECURITY.md` | Security policy & responsible disclosure |
| `PRIVACY.md` | Privacy policy |
| `CITATION.cff` | How to cite EDNA |

## Verification

The deterministic vectors and reference results in `verification/` are the public evidence base. The engine itself ships exclusively as a **signed, obfuscated bundle** bound to licensed domains; licensees can replay the vectors against their licensed bundle. Field-test participation follows `verification/FIELD-TEST.md` — reports are submitted as GitHub issues in this repository.

## Links

- Product & live engine: [openos.space](https://openos.space)
- Login with OpenOS (passwordless EDNA ID SDK): [openos.space/developers/login](https://openos.space/developers/login)
- License text: [EPCAL-1.0](https://openos.space/EPCAL-1.0.txt)

© Sebastian Kläy (Sebklaey). All rights reserved.
