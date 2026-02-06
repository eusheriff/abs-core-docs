# ABS MCP Gateway Contract V1.0

## Overview
This document defines the interface between the ABS MCP Gateway and the ABS Policy Engine/Clients. The Gateway serves as an MCP Server that intercept tool calls, maps them to ABS Intents, and enforces policies before execution.

## 1. Intent Mapping (MCP -> ABS)
When `CallToolRequest` is received:

```typescript
// MCP Request
{
  method: "tools/call",
  params: {
    name: "trade_executed",
    arguments: { symbol: "BTC", amount: 1 }
  }
}

// Maps to ABS Intent
{
  request_id: "uuid-v4",
  timestamp_utc: "ISO-8601",
  actor_id: "mcp-client-id",
  intent_type: "TOOL_CALL", // or mapped from metadata
  intent_payload: {
    tool_name: "trade_executed",
    arguments: { symbol: "BTC", amount: 1 }
  },
  canonical_intent_hash: "sha256(...)",
  policy_bundle_id: "determined-by-config",
  evidence_manifest: { ... } // gathered from context
}
```

## 2. Receipt Structure (ABS -> MCP)
The Gateway returns a standard `PolicyDecisionReceipt` signed with Ed25519.

## 3. Interfaces

### Intent
```json
{
  "request_id": "string",
  "timestamp_utc": "string",
  "actor_id": "string",
  "intent_type": "string",
  "intent_payload": "object",
  "canonical_intent_hash": "string",
  "auth_signature": "string",
  "policy_bundle_id": "string",
  "evidence_manifest": "object"
}
```

### Receipt
```json
{
  "receipt_id": "string",
  "decision": "APPROVED | DENIED",
  "receipt_payload_hash": "string",
  "receipt_signature": "string (Ed25519)",
  "public_key_id": "string",
  ...
}
```

## 4. Latency Requirement
- **Goal**: P95 <= 10ms for policy evaluation (excluding tool execution).
- **Implementation**: In-process integration with `@oconnector/abs-core`.
