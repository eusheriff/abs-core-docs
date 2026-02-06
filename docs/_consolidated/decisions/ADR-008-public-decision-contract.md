# ADR-008: Public Decision Contract (Decision Envelope v1)

**Status:** Accepted for v3.0.0
**Date:** 2026-01-22
**Context:** Commercial adoption of ABS Kernel requires a defensible, audit-proof "Decision Object". Previous formats lacked versioning, explicit authority, and clear separation between governance intents and system errors. Legal review emphasized the need for standardized Applicability Checks, strict Monitor Mode enforcement, and clear Jurisdiction boundaries.

## Decision
We enforce the **Decision Envelope v1** as the immutable output contract of the ABS Kernel. All clients (CLI, SDK, API) MUST assume this schema.

### The Contract (Schema)
```typescript
interface DecisionEnvelope {
  // --- 1. Contract Metadata (Version Control) ---
  contract_version: "1.0.0";   // Semantic version of this schema
  
  // --- 2. Identity & Integrity (Tamper Evidence) ---
  decision_id: string;         // UUID v4 (Unique Identifier)
  trace_id: string;            // Correlation ID for distributed tracing
  timestamp: string;           // ISO-8601 (When the decision was finalized)
  valid_until?: string;        // ISO-8601 (Decision Expiry - Temporal Validity)
  
  signature: {
    alg: "HMAC-SHA256";        // Algorithm used
    key_id: string;            // Key Identifier (for rotation support)
    value: string;             // The seal (Hex)
  };
  // Note: HMAC provides Integrity & Authenticity. Non-repudiation requires Asymmetric Signatures.

  // --- 3. The Verdict (The What) ---
  decision_type: "GOVERNANCE" | "SYSTEM_ERROR"; // Differentiates Policy vs Crash
  verdict: Verdict;

  // --- 4. The Rationale (The Why - Legal Defense) ---
  reason_code: ReasonCode;     // Machine-readable enum (Stable)
  reason_human: string;        // Human-readable explanation
  risk_score: number;          // 0-100 (Absolute Scale: 0=Safe, 100=Critical)
  
  // --- 5. Authority (The Who) ---
  authority: {
    type: "POLICY" | "HUMAN" | "SYSTEM"; // Nature of the judge
    id: string;                          // Policy Name, User ID, or System Component
    delegation_chain?: string[];         // Trace of inheritance
  };

  // --- 6. Governance Context (The Scope) ---
  jurisdiction?: string;       // Format: "^[A-Z]{2}-[A-Z0-9_]+$". Default: "CORP-INTERNAL"
  policy_id: string;           // ID of the specific logic applied
  policy_version?: string;     // Hash/Version of the policy logic (Reproducibility)
  monitor_mode: boolean;       // If true, verdict was simulated (Shadow Mode)

  // --- 7. Operational Actions ---
  applicability?: {
      required_checks: ApplicabilityGate[]; // Standardized gates
  };
  constraints?: string[];      // e.g. ["read-only", "sandbox", "audit-log-required"]
  required_actions?: string[]; // e.g. ["human_approval_request", "mfa_challenge"]
}

enum Verdict {
  // Governance States (Only valid when decision_type = GOVERNANCE)
  ALLOW = 'ALLOW',
  DENY = 'DENY',
  REQUIRE_APPROVAL = 'REQUIRE_APPROVAL',
  ALLOW_WITH_CONSTRAINTS = 'ALLOW_WITH_CONSTRAINTS',
  
  // Error States (Only valid when decision_type = SYSTEM_ERROR)
  SYSTEM_FAILURE = 'SYSTEM_FAILURE' 
}

enum ApplicabilityGate {
  JURISDICTION_MATCH = 'JURISDICTION_MATCH', // Scope validation
  INCIDENT_CLEAR = 'INCIDENT_CLEAR',         // No critical overrides
  TENANT_ACTIVE = 'TENANT_ACTIVE',           // Subscription/State valid
  POLICY_ACTIVE = 'POLICY_ACTIVE'            // Policy revocation check
}

enum ReasonCode {
  // --- Access Control (Identity) ---
  AUTH_INVALID = 'AUTH.INVALID',
  AUTH_SCOPE_MISSING = 'AUTH.SCOPE_MISSING',
  
  // --- Policy & Risk (Logic) ---
  POLICY_VIOLATION = 'POLICY.VIOLATION',
  RISK_THRESHOLD_EXCEEDED = 'RISK.EXCEEDED',     // Score > Threshold
  SEQUENCE_VIOLATION = 'SEQUENCE.VIOLATION',     // Bad Pattern Detected
  
  // --- Integrity (Security) ---
  INPUT_INJECTION = 'INPUT.INJECTION',           // Prompt Injection
  INPUT_MALFORMED = 'INPUT.MALFORMED',           // Schema Failure
  INTEGRITY_TAMPERING = 'INTEGRITY.TAMPERED',    // Hash Chain Broken
  
  // --- Operational (System) ---
  RATE_LIMIT_EXCEEDED = 'OPS.RATE_LIMIT',
  SYSTEM_MAINTENANCE = 'OPS.MAINTENANCE',
  INTERNAL_ERROR = 'OPS.INTERNAL_ERROR'
}
```

### Invariants & Normative Rules (Binding)
Violating any invariant below MUST result in rejection at generation time.

1.  **State Consistency**: 
    - **IF** `decision_type` IS `GOVERNANCE`, **THEN** `verdict` MUST NOT be `SYSTEM_FAILURE`.
    - **IF** `decision_type` IS `SYSTEM_ERROR`, **THEN** `verdict` MUST be `SYSTEM_FAILURE`.
2.  **Risk Semantics**: `risk_score` semantics are strictly defined by `policy_version`. Scores are comparable ONLY within the same policy version. **IF** `risk_score` is used for comparison, `policy_version` MUST be present.
3.  **Monitor Mode Safety**: **IF** `monitor_mode` IS `true`, the envelope IS **INVALID** for authorization. Execution systems MUST treat it as a non-binding simulation.
4.  **Applicability Checks**: Execution systems MUST validate `applicability.required_checks` and emit a valid **ExecutionReceipt**.
5.  **Domain Authority**: `jurisdiction` MUST follow the format `^[A-Z]{2}-[A-Z0-9_]+$` (e.g., "US-HIPAA"). If undefined, implies "CORP-INTERNAL".

### ExecutionReceipt – Normative Requirements (v1.1)
An ExecutionReceipt MUST:
1.  Reference `decision_id` (foreign key) from the Envelope.
2.  Include a unique `receipt_id` and `timestamp` (ISO-8601).
3.  Include **Executor Identity** (`executor_id`) and **Execution Context** (Region/Env/Tenant).
4.  Record all gate checks with detailed evidence (`checked_at`, `source`, `result`).
5.  Be generating using a synchronized time source (NTP). Detected clock skew > threshold MUST be logged as non-compliance.
    
### Authority & Lifecycle Boundaries
1.  **Authority Expiry (Implicit vs Explicit)**: 
    - **IF** `valid_until` is defined, any `ExecutionReceipt` with `timestamp > valid_until` MUST be considered **INVALID** and non-compliant.
    - **IF** `valid_until` is undefined, authority is strictly limited to the **same request/transaction boundary** in which it was issued. Any deferred, queued, or retried execution requires an explicit `valid_until`.
2.  **Gate Enforcement & SKIPPED Semantics**:
    - `JURISDICTION_MATCH` and `TENANT_ACTIVE` **MUST NOT** be `SKIPPED`.
    - Other gates MAY be `SKIPPED` only if explicitly allowed by `policy_version`.
    - **Rule**: A `SKIPPED` gate indicates lack of evidence and MUST be treated as non-authorizing unless explicitly allowed.
3.  **Fact Freshness & Evidence**:
    - Applicability checks MUST be evaluated against **authoritative data sources** at execution time. Cached evaluations are non-compliant unless explicitly permitted.
    - Gates SHOULD include evidence metadata: `checked_at` (ISO-8601) and `source` (System ID).
4.  **Receipt Cardinality**:
    - Multiple `ExecutionReceipts` MAY reference the same `decision_id` only if each represents a distinct execution attempt.
    - Each attempt MUST independently satisfy authority, applicability, and validity constraints.
5.  **Liability & Epistemic Boundary**:
    - The ABS Kernel is authoritative for **decision issuance only**. 
    - **Epistemic Guarantee**: The Kernel guarantees the integrity of decisions based on facts as evaluated at issuance and verified at execution. It does **not** guarantee the continued truth of external facts.
    - Responsibility for execution outside the envelope’s `valid_until`, `jurisdiction`, or `applicability` constraints lies entirely with the executor.

> **Non-Compliance**: Failure to emit a valid receipt constitutes a breach of the governance contract, shifting liability to the executor.

## Consequences
1.  **Legally Defensible**: Creates an audit-proof trail of *intent* and *authority*, defensible within the agreed governance framework.
2.  **Operational Safety**: `monitor_mode` and `applicability` gates prevent accidental misuse of valid decisions in invalid contexts.
3.  **Audit Completeness**: The requirement for an `ExecutionReceipt` closes the loop between "Decision Issued" and "Decision Enforced".
4.  **Audit Reproducibility**: `policy_version` allows auditors to load the *exact* logic enabled at the time of decision.

## Status
ACCEPTED.
