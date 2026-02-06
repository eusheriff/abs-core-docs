# Security Architecture

## 1. Threat Model & Mitigations
ABS Core is designed to mitigate the **OWASP Top 10 for LLM Applications**.

### LLM01: Prompt Injection
**Risk**: Malicious user input manipulating the LLM instruction.
**Defense**: Strict separation of `system` vs `user` messages and input sanitization.
**Code**: See [`Sanitizer.sanitize()`](packages/core/src/core/sanitizer.ts).

### LLM02: Insecure Output Handling
**Risk**: LLM generating XSS or SQL injection payloads.
**Defense**: We strictly type LLM outputs to JSON schemas and do not execute code directly.
**Code**: See [`EventProcessor.process()`](packages/core/src/core/processor.ts) (Uses JSON mode).

### LLM08: Excessive Agency
**Risk**: LLM taking actions autonomously without oversight.
**Defense**: The **Policy Gate** is a hard constraint. No action is executed without an `ALLOW` decision log.
**Code**: See `PolicyEngine` in [`packages/core/src/core/dynamic-policy.ts`](packages/core/src/core/dynamic-policy.ts).

## 2. Infrastructure Security
- **Cloudflare**: DDoS protection and WAF (Rate Limiting) enabled by default.
- **D1 Database**: At-rest encryption (Cloudflare managed).
- **Secrets**: API Keys managed via `wrangler secret` (encrypted environment variables).

## 3. Auditability
- **Immutable Logs**: Every decision is logged *before* execution.
- **Traceability**: `trace_id` propagates from Event -> LLM -> Log -> Action.

## 4. OWASP Top 10 Coverage Matrix (v2.x)

| ID | Risk | Mitigation in ABS Core | Implementation |
| :-- | :-- | :-- | :-- |
| **LLM01** | **Prompt Injection** | Input sanitization + pattern detection | `src/core/sanitizer.ts` |
| **LLM02** | **Insecure Output Handling** | Outputs must pass Policy Engine | `src/core/processor.ts` |
| **LLM08** | **Excessive Agency** | Human-in-the-Loop defaults, API Scopes | `src/api/middleware/auth.ts` |
| **LLM09** | **Overreliance** | Assume LLM is wrong until proven otherwise | Immutable Audit Log (Hash Chain) |

## 5. Supported Versions

| Version | Supported | Notes |
| :-- | :-- | :-- |
| v2.0.x | :white_check_mark: | Scale Release (Current) |
| v1.0.x | :white_check_mark: | Enterprise Trust Release |
| v0.9.x | :x: | End of Life |

## 6. Reporting a Vulnerability

Please report sensitive security issues via GitHub Security Advisories on this repository.
**DO NOT** open public issues for exploits.
