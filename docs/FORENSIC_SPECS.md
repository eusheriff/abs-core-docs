# ABS Forensic Log Specification (v1.1)

**Version**: 1.1.0
**Authority**: ABS_INSTITUTIONAL_GATEKEEPER
**Mode**: ALL_OR_NOTHING
**Fail-Closed**: true

## 1. Core Principles

1.  **JSONL Format**: One event per line, Append-Only.
2.  **Tamper-Evident**: SHA-256 Hash Chaining.
3.  **Non-Repudiation**: Ed25519 Signatures.
4.  **Offline Verification**: Self-contained except for Trust Anchors (Root of Trust).
5.  **Strict Canonicalization**: No floats, sorted keys, no whitespace.

## 2. Event Schema

```typescript
interface ForensicEvent {
  // --- Integrity Headers ---
  /** Unique ID (UUID v4) */
  id: string;
  /** ISO 8601 Timestamp (UTC) */
  timestamp: string;
  /** Sequence number (0-indexed, strictly increasing) */
  sequence: number;
  /** SHA-256 Hash of PREVIOUS canonical event (Genesis = 64 zeros) */
  prev_hash: string;
  
  // --- Polymorphic Payload ---
  /** Event Type Enum */
  type: ForensicEventType;
  /** Actor performing the action */
  actor_id: string;
  /** Typed payload matching the Type contract */
  payload: Record<string, any>;
  
  // --- Security ---
  /** ID of the Key in the Registry */
  public_key_id: string;
  /** The public key used (Hex) for immediate verification */
  public_key: string;
  /** Ed25519 Signature of Canonical(Event w/o signature) */
  signature: string;
}
```

## 3. Root of Trust & Genesis

The first event (`sequence: 0`) **MUST** be `SYSTEM_START` with strict payload:
- `prev_hash`: `"0000000000000000000000000000000000000000000000000000000000000000"`
- `payload.kernel_version`: Semver string.
- `payload.build_hash`: Integrity hash of the runtime.
- `payload.environment`: "production" | "staging" | "dev".

## 4. Key Lifecycle

Key events are first-class citizens to allow rotation audit.
- `KEY_REGISTER`: Minting a new key identity.
- `KEY_ROTATE`: Retiring old, authorizing new.
- `KEY_REVOKE`: Emergency kill-switch.

## 5. Event Types & Contracts

| Type | Description | Required Fields |
| :--- | :--- | :--- |
| `SYSTEM_START` | Boot | kernel_version, build_hash |
| `SYSTEM_STOP` | Shutdown | exit_code, reason |
| `TOOL_CALL` | Agent Action | tool_name, args, idempotency_key |
| `ACTION_CALL` | Side Effect | action_name, params |
| `DECISION` | Governance | decision, severity, policy_bundle_id |
| `KEY_REGISTER` | IAM | public_key_id, scope |

## 6. Canonicalization Rules (RFC 8785 subset)

1.  **Lexicographical Sort**: All keys sorted.
2.  **No Whitespace**: `JSON.stringify` with no padding.
3.  **No Floats**: Numbers must be Integers or Strings. 
    *   Valid: `100`, `"100.50"`
    *   Invalid: `100.5` (native float) - *Must be rejected by serializer*
4.  **Signature Exclusion**: The `signature` field is removed before canonicalization for signing/hashing.

## 7. Verification Algorithm

1.  Genesis check (Seq 0, PrevHash 0, Payload valid).
2.  For each event $E_n$:
    *   Assert $E_n.prev\_hash == H(E_{n-1})$.
    *   Assert $Verify(Sig_{n}, PubKey_{n}, Canonical(E_n)) == True$.
    *   Assert Key valid for time $T_n$ (not revoked).
    *   Assert Sequence $S_n == S_{n-1} + 1$.
3.  Fail Closed if ANY check fails.

## 8. Artifacts
- `log.wal`: The forensic log.
- `evidence/forensic_proof.json`: The output of a successful verification run.
