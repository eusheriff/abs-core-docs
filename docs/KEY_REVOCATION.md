# Key Revocation Procedure

## Trigger
Revocation is triggered when a private key is leaked or suspected compromised.

## Steps
1. Identify the `id` of the compromised key.
2. Update `keys/public_keys.json`: set `status: REVOKED`.
3. Sign registry with offline root key.
4. Push update immediately to all enforcement points (CLI, Gateways).
5. All receipts signed by this key will immediately fail verification.

## Emergency Mode
If the Root Key is compromised, a full ABS infrastructure reset is required (Re-init).
