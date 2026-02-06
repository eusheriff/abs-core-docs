# Key Rotation Procedure

## Overview
ABS uses Ed25519 keys for signing receipts. Keys must be rotated annually or upon compromise.

## Lifecycle Statuses
- **ACTIVE**: Key is used for signing and verifying.
- **REVOKED**: Key is compromised. Receipts signed by this key are invalid regardless of date.
- **EXPIRED**: Key is no longer used for new signatures but remains valid for verifying historical receipts (within window).

## Rotation Steps
1. Generate new Ed25519 keypair.
2. Add new key to `keys/public_keys.json` with `status: ACTIVE` and future `valid_from`.
3. Update old key `valid_until` to overlap (soft rotation) or cut off (hard rotation).
4. Sign the registry with the *offline institutional key*.
5. Distribute new `public_keys.json` + `.sig`.

## Verification Logic
1. Load `public_keys.json`.
2. Find key matching `receipt.key_id` (not currently implemented, assumes single key or iterative check).
3. Check `status != REVOKED`.
4. Check `valid_from <= receipt.timestamp <= valid_until`.
