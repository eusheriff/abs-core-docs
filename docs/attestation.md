# ABS Context Attestation Tokens (CAT)

> **Protocol Version**: v2.0
> **Status**: Stable (Phase 4.1)

## Overview

ABS Context Attestation Tokens (CAT) extend the v1 AAT protocol to include **Cognitive Context**. This creates a "Zero Trust" binding between the agent's permission and its internal state (Memory & Reputation).

If an agent's memory is tampered with (e.g., via Prompt Injection), the `memoryDigest` will change, potentially invalidating cached CATs or triggering downstream rejection if the recipient validates state continuity.

## Token Format

CAT follows the compact, URL-safe format:

```
v2.<PAYLOAD_BASE64URL>.<SIGNATURE_HEX>
```

### Payload Structure

The payload is a canonicalized JSON object containing:

| Field | Key (Short) | Type | Description |
|---|---|---|---|
| Verdict | `v` | string | `ALLOW`, `DENY` |
| Agent ID | `a` | string | `metadata.agentId` |
| Tool | `t` | string | Tool Name |
| Risk | `r` | number | Static Risk Score |
| Timestamp | `ts` | number | Epoch ms |
| Policy Hash | `ph` | string | Config Version |
| Audit ID | `id` | string | Audit Log ID |
| **Memory Digest** | `md` | string | Hash of Agent Context (v2) |
| **Reputation** | `rep` | number | Adaptive Risk Score (v2) |
| **Nonce** | `n` | string | Anti-replay (v2) |
| **Audience** | `aud` | string | Target Service ID (v2.1) |

## Verification (Offline)

Standard Ed25519 verification applies.

```typescript
import { AttestationSigner } from '@abs/core';
// Verify
const payload = await AttestationSigner.verify(tokenStr, publicKeyHex);
if (payload.reputationScore < 50) {
  throw new Error("Agent Reputation too low!");
}
```

## Security Guarantees

1.  **Non-Repudiation**: The ABS Kernel cannot deny having authorized the action.
2.  **Anti-Forgery**: Agents cannot generate valid tokens without the Kernel's private key.
3.  **Replay Protection**: The `ts` (timestmap) and `id` (audit ID) can be used to prevent replay attacks (max validity window recommended: 60s).
