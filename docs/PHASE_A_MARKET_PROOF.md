# Phase A: Market Proof (The Inevitability Strategy)

> **Objective:** Provar publicamente que o ABS bloqueia comportamentos reais perigosos e que terceiros conseguem reproduzir isso sem ajuda.

## Mandate (from `ABS_PHASE2_EXECUTION_PROMPT_V1`)

- **Context:** ABS Kernel, Gate V1 Passed.
- **Goal:** Market Inevitability via "Receipts or it didn't happen".
- **Rule:** Phase A selected. ALL_OR_NOTHING.
- **Constraints:**
  - No internal validation only.
  - Proof must be external and reproducible.
  - Fail closed default.

## Deliverables

### 1. Public Demo Repo (`public_demo_repo`)
- **Action:** Create a clean, standalone repository structure (simulated in `examples/public-demo` or similar) that a user can clone.
- **Content:**
  - `package.json` (minimal).
  - `abs.config.yaml` (pointing to `ABS_DEFAULT_L1_SAFE_MODE`).
  - `demo.ts` (The attack script).
  - `README.md` (Step-by-step reproduction instructions).

### 2. Deny Demo Script (`deny_demo_script`)
- **Action:** Refine `tests/ttfv_deny_demo.ts` into a customer-facing script `examples/public-demo/attack_simulation.ts`.
- **Logic:**
  - Attempts `rm -rf /` via MCP or pure CLI.
  - Expects `DENIED` receipt.
  - Prints "✅ ABS Protected You" upon success.

### 3. Offline Receipt Verification (`offline_receipt_verification`)
- **Action:** Package the receipt verification logic into a standalone tool/script `examples/public-demo/verify_receipt.ts`.
- **Value:** Proves that the denial came from a signed, trusted kernel, not just a mocked catch block.

### 4. CI Blocking Example (`CI_blocking_PR_example`)
- **Action:** Enhance `.github/workflows/abs-verify-receipts.yml` to be a copy-pasteable example of how to gate PRs on ABS receipts.
- **Output:** `examples/public-demo/.github/workflows/ci-gate.yml`.

### 5. Visible Badge (`README_badge_visible`)
- **Action:** Update the main `README.md` to feature the "Protected by ABS" badge prominently, pointing to the public demo.

## Success Metrics

1.  **Time-to-First-Violation (TTFV):** <= 60 seconds (Target: < 5s).
2.  **Install Time:** <= 5 minutes (for a 3rd party).
3.  **Reproducibility:** 100% without author intervention.

## Execution Plan

1.  **Scaffold Public Demo**: Create `examples/public-demo` structure.
2.  **Package Scripts**: Move and clean up `ttfv_deny_demo.ts` and `verify_receipt_offline.ts`.
3.  **Documentation**: Write `examples/public-demo/README.md`.
4.  **Verification**: Run the full "3rd party user" flow locally to measure timing.
