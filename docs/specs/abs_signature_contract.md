# ABS Zero-Trust Signature Contract (WebAuthn/COSE)
## Protocol Specification v1.0

### 1. Abstract
This document defines the JSON data structures exchanged between the **ABS Hypervisor** (The Challenger) and the **Human Authenticator** (The Signer) to authorize high-risk AI actions. This protocol ensures **Non-Repudiation** by cryptographically binding the *Semantic Intent* of the AI to the *Biometric Authorization* of the Human.

### 2. The Flow
1.  **Intent**: AI proposes `Action X`.
2.  **Challenge**: Hypervisor constructs `TxChallenge` and hashes it.
3.  **Sign**: Human Mobile Device signs the hash using WebAuthn (Secure Enclave).
4.  **Verify**: Hypervisor validates signature against the Admin's Public Key.

### 3. Data Structures

#### 3.1. Transaction Challenge (The Payload)
The "What" that is being signed. Canonical serialization is required before hashing.

```json
{
  "protocol": "ABS-V3-ZTS",
  "version": "1.0",
  "chain_id": "abs-mainnet-1",
  "nonce": "random_hex_32_bytes",
  "timestamp": 1700000000,
  "intent": {
    "agent_id": "agent-bravo",
    "tool_name": "stripe_transfer",
    "parameters": {
      "amount": 50000,
      "currency": "USD",
      "destination": "acct_CaymanIslands"
    },
    "risk_score": 85,
    "risk_reason": "Value > $10000 threshold"
  }
}
```

#### 3.2. WebAuthn Assertion (The Proof)
The standard W3C WebAuthn response structure.

```json
{
  "id": "credential_id_base64",
  "rawId": "raw_credential_id_base64",
  "type": "public-key",
  "response": {
    "authenticatorData": "auth_data_base64",
    "clientDataJSON": "client_data_json_base64",
    "signature": "signature_base64",
    "userHandle": "user_handle_base64"
  },
  "abs_meta": {
    "signer_id": "ciso-admin-001",
    "algo": "ES256" // ECDSA w/ P-256 and SHA-256
  }
}
```

### 4. Verification Logic (Rust Hypervisor)
The Hypervisor must perform the following checks:
1.  **Parse** `clientDataJSON` to verify the `challenge` matches `SHA256(TxChallenge)`.
2.  **Verify** `authenticatorData` flags (User Present / User Verified).
3.  **Verify** the ECDSA signature over the `authData + clientDataHash` using the stored Public Key.

### 5. Fallback (Ed25519)
For non-WebAuthn clients (e.g., CLI tools), a raw Ed25519 signature of the `SHA256(TxChallenge)` is accepted if the key is registered as type `ED25519_RAW`.

```json
{
  "signer_id": "ciso-admin-cli",
  "algo": "Ed25519",
  "signature": "hex_encoded_signature",
  "signed_hash": "sha256_of_challenge"
}
```
