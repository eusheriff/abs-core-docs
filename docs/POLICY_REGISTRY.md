# Policy Registry

## Overview
The Policy Registry ensures that the ABS Policy Engine only loads trusted, versioned policy bundles.

## Structure
- `docs/policies/index.json`: The canonical list of trusted policies.
- `docs/policies/index.sig`: The detached Ed25519 signature of `index.json`.

## Trust Model
1. **Engine Pin**: The index pins the `engine_version` (e.g. `1.1.0`). If the running engine version mismatch, it moves to Safe Mode using local fallbacks.
2. **Bundle Verification**: Each bundle in `policies` has a `sha256` hash. The engine computes the hash of the loaded `.yaml` and rejects it if it doesn't match the index.
3. **Index Integrity**: At startup, the engine verifies `index.sig` against the Active Key in `keys/public_keys.json`.

## Maintenance
To add or update a policy:
1. Edit the `.yaml` file in `docs/policies/`.
2. Run `npx tsx scripts/generate_policy_index.ts`.
3. Sign the new index: `npx tsx scripts/sign_file.ts docs/policies/index.json`.
4. Commit both files.
