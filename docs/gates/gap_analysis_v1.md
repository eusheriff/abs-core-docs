# Gap Analysis: ABS Indispensable Gate V1

**Date**: 2026-01-28
**Gate ID**: `ABS_INDISPENSABLE_GATE_V1`

## Status Summary

| Gate | Description | Status | Missing Artifacts |
|------|-------------|--------|-------------------|
| **G1** | TTFV <= 60s Block/Verify | 🟡 PARTIAL | `evidence/ttfv_timestamps.json`, `receipts/deny_demo.json` |
| **G2** | `abs report` command | 🔴 FAIL | Command missing. Reports missing. |
| **G3** | External Inevitability (CI/Badge) | 🟡 PARTIAL | `docs/BRANCH_PROTECTION.md`, Badge Generator. |
| **G4** | Key Lifecycle (Rotate/Revoke) | 🔴 FAIL | `docs/KEY_ROTATION.md`, `public_keys.json`. |
| **G5** | Policy Registry (Signed/Pinned) | 🔴 FAIL | `docs/policies/index.json`, `index.sig`. |

## Plan of Action

1.  **Implement `abs report` (G2)**:
    - Create `ReportService` in CLI.
    - Output formats: MD, JSON, CSV.
    - Analytics: Decision counts, severity, trace IDs.

2.  **Formalize Key Lifecycle (G4)**:
    - Define `public_keys.json` registry format.
    - Create `docs/KEY_ROTATION.md` (Procedure).
    - Implement revocation check in `abs verify`.

3.  **Policy Registry (G5)**:
    - Create `index.json` generation script (`abs-docs-gen`).
    - Sign index with `ABS_SECRET_KEY` (or dedicated key).

4.  **Inevitability Pack (G3)**:
    - Write `docs/BRANCH_PROTECTION.md`.
    - Add Badge generation to `abs report` or separate command.

5.  **Final Verification (G1)**:
    - Run E2E "Deny Demo".
    - Collect all evidence in `evidence/`.
