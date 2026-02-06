# Branch Protection Strategy

To ensure ABS Inevitability, repository settings must prevent code from merging without valid receipts.

## Implementation
1. **Require Status Checks**:
   - Enable "Require status checks to pass before merging".
   - Select `verify` (from `abs-verify-receipts.yml`).

2. **Block Force Pushes**:
   - Enable "Restrict force pushes" to preserve the `audit.jsonl` history on main/master.

3. **Require Signed Commits (Optional but Recommended)**:
   - Align Git signing with ABS Receipt signing.

## Badge for README
Use the generated badge to signal enforcement status:
`![Governance: Enforced](evidence/ci/badge_sample.svg)`
