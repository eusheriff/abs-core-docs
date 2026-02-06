# ABS Gate Rules: Ops Governance
> **Policy Set**: OPS_GOV_v1
> **Target**: Clawdbot Intents

## Fail-Closed Matrix

If an intent fails any check, the default response is `DENY` with a specific error code.

| Check | Failure Condition | Action | Error Code |
| :--- | :--- | :--- | :--- |
| **Schema Validation** | Field missing, wrong type | DENY | `INVALID_SCHEMA` |
| **Authentication** | Invalid signature/token | DENY | `AUTH_FAILED` |
| **Replay Protection** | `timestamp` < `now - ttl` OR `nonce` seen | DENY | `REPLAY_DETECTED` |
| **Policy Scope** | `actor_id` not in `policy.allowed_actors` | DENY | `UNAUTHORIZED_ACTOR` |
| **State Guard** | Action invalid for current system state (e.g. Pause when already Paused) | DENY | `INVALID_STATE_TRANSITION` |

## Policy Mapping

| Action | R-Level | Requirement |
| :--- | :--- | :--- |
| `OPS_STATUS` | **R0** | None. Rate limit: 60/min. |
| `OPS_REPORT` | **R0** | Rate limit: 5/min. |
| `OPS_PAUSE` | **R1** | Reason encouraged. Logs to audit. |
| `OPS_RESUME` | **R2** | **REQUIRE_APPROVAL** (2-step). Reason required. |
| `OPS_CONFIG` | **R3** | **REQUIRE_APPROVAL** + Admin Role. |
| `OPS_PANIC` | **R3** | **IMMEDIATE**. Bypass latch. Triggers `SEV-1`. |

## Rate Limits

Applied per `actor_id` to prevent DoS or accidental spam.

*   `scope:read`: 60 requests / minute
*   `scope:control`: 10 requests / minute
*   `scope:admin`: 5 requests / hour
*   `scope:critical`: 1 request / minute (burst 1)

## Environment Scoping

*   **Production**: Strict enforcement. All R2/R3 require explicit confirmation.
*   **Staging**: R0-R2 allowed freely. R3 warns.
*   **Dev**: All allowed.
