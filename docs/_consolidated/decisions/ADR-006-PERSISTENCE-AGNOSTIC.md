# ADR-006: Persistence Agnostic Architecture (Async Interface + KV)

## Status
ACCEPTED

## Context
The previous persistence layer was tightly coupled to synchronous SQLite queries (`better-sqlite3`), preventing deployment to Edge environments (Cloudflare Workers) and creating a single point of failure. Additionally, `RiskTelemetry` relied on in-memory storage, making agent reputation volatile and vulnerable to process crashes (Amnesia Attack).

## Decision
We refactored the kernel to use an agnostic, asynchronous `IPersistenceLayer` interface.

1.  **Interface**: Defined `IPersistenceLayer` in `@abs/policy` enabling `kv` (Key-Value) and relational operations (`execute`, `query`).
2.  **Adapters**:
    - `D1Adapter` (Cloudflare D1) for Production/Edge.
    - `SQLiteAdapter` (Refactored) for Local Development/Testing.
3.  **Risk Persistence**: Migrated `RiskTelemetry` to use `IPersistenceLayer.kv` for durable reputation tracking.
4.  **Async Core**: Updated `ABSPolicyEngine`, `RiskTelemetry`, and `Server` to use async persistence patterns.

## Consequences
### Positive
- **Edge Compatible**: System can now run on Cloudflare Workers/D1.
- **Security**: Risk scores persist across restarts (Fixes Amnesia Vulnerability).
- **Flexibility**: Can swap backends (e.g., Turso, Postgres) by implementing `IPersistenceLayer`.

### Negative
- **Complexity**: Async everywhere requires careful handling of promises in what was previously sync logic.
- **Breaking Change**: Legacy `LocalDBAdapter` and synchronous `db.exec` calls were removed/refactored.

## References
- RFC-001 (Persistence Refactoring)
- AUDIT_REPORT_2026_02_04.md
