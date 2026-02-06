# ADR-007: ABS Hypervisor (MCP Security Proxy)

## Status
ACCEPTED

## Context
Traditional AI governance relies on "cooperative" agents asking for permission (`abs.verify()`).
This model is fragile: authorized autonomous agents (or compromised ones) can bypass governance by invoking tools directly via the Model Context Protocol (MCP).

## Decision
We will implement **ABS Hypervisor**, a "Man-in-the-Middle" security proxy for MCP servers.
- **Architecture**: Stdio/HTTP Interceptor.
- **Role**: Sits transparently between the Agent (Client) and the Tool (Server).
- **Mechanism**: Inspects JSON-RPC 2.0 messages. Blocks `tools/call` based on real-time `RiskTelemetry`.
- **Enforcement**: Physical firewall. If `Risk > Threshold`, the message is dropped, and a fake error is returned to the Agent.

## Consequences
### Positive
- **Zero-Trust**: Governance is enforced even if the agent code is malicious/compromised.
- **Real-Time**: Integrates with Session 16's Threat Intel push model for instant reaction.
- **Legacy Compat**: Works with any existing MCP Server (Notion, Filesystem, Postgres) without code changes.

### Negative
- **Latency**: Adds small overhead (<10ms) per tool call.
- **Complexity**: Debugging tool failures requires checking Proxy audit logs.
