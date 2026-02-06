# ABS V6.0: Sovereign Identity Protocol (Agent Passports)
**Status**: STANDARD
**Objective**: Identity Emission & Verification for Autonomous Agents.

---

## 1. The "Agent Passport" (JSON-LD)
The agent carries a cryptographic proof that it has been trained, tested, and is governed by an ABS Hypervisor. This serves as the "Sovereign Identity" for the agent economy.

```json
{
  "@context": "https://www.w3.org/ns/did/v1",
  "id": "did:abs:manu-001",
  "verificationMethod": {
    "id": "did:abs:manu-001#key-1",
    "type": "Ed25519VerificationKey2020",
    "controller": "did:abs:governance-root",
    "publicKeyMultibase": "z6Mkm9k..."
  },
  "service": [{
    "id": "abs-compliance-audit",
    "type": "AuditLog",
    "serviceEndpoint": "https://audit.abs.io/manu-001"
  }],
  "abs_claims": {
    "risk_profile": "LOW",
    "certified_by": "ABS-Hypervisor-V6",
    "last_compliance_sync": 1738778400
  }
}
```

## 2. Verification Protocol
When `Agent A` connects to `Agent B`:

1.  **Handshake**: Agent A presents its DID Passport.
2.  **Challenge**: Agent B sends a `nonce`.
3.  **Proof**: Agent A signs the nonce with its `Ed25519` private key (sealed in TEE).
4.  **Validation**: Agent B checks the signature and the `certified_by` claim against the Global Reputation Ledger.

## 3. Revocation (The Kill List)
The `did:abs:governance-root` can issue a Revocation Claim on the Ledger.
*   If an agent is compromised, its Passport is burned.
*   All ABS nodes globally reject connections from that DID instantly.
