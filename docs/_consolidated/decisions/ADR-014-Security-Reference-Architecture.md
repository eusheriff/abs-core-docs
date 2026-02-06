# ADR-014: ABS Security Reference Architecture

## Status
Accepted

## Context
The ABS ecosystem required a robust security architecture to manage autonomous agent risks effectively. Key challenges included preventing secret leakage (INV-001), ensuring memory authority limits (INV-004), and providing non-repudiation for audit logs (INV-006).

## Decision
We implemented a multi-layered security architecture:

1.  **Risk Telemetry:** A sliding window aggregation service (`RiskTelemetryService`) that calculates risk scores and automatically escalates autonomy levels (L0-L3).
2.  **Circuit Breakers:** A `CircuitBreakerRegistry` managing per-capability failure thresholds to fail-closed during instability.
3.  **Secret Sanitization:** A proactive `SecretSanitizer` enforcing INV-001 by scanning all LLM inputs/outputs for 20+ secret patterns.
4.  **MemU Bridge:** A safe adapter enforcing INV-004 by tagging all memory outputs as `UNTRUSTED` and `ADVISORY`, preventing memory from escalating privileges.
5.  **Ed25519 Signing:** Migration from HMAC-SHA256 to Ed25519 asymmetric keys for non-repudiation and third-party verification of audit logs.

## Consequences
- **Positive:** System is now fail-closed by default. Audit logs are cryptographically verifiable by third parties. Agents have adaptive autonomy based on real-time risk.
- **Negative:** Increased complexity in signature management. Slight performance overhead from sanitization regex scanning.
