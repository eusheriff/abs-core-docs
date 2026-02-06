# Hardening Checklist: Ops-Gov-Exec Triad

## Threat Model (Focus: Telegram/Gateway)

1.  **Account Takeover (Telegram)**
    *   **Risk**: Attacker gains access to Operator's Telegram.
    *   **Mitigation**:
        *   R2/R3 commands require a secondary "Challenge Phrase" or 2FA via different channel/app.
        *   Allowed IDs hardcoded, not dynamic.
        *   "Panic Mode" accessible to multiple admins (redundancy).

2.  **Gateway Compromise (Clawdbot Node)**
    *   **Risk**: Attacker RCE on Clawdbot server.
    *   **Mitigation**:
        *   Clawdbot has NO keys. It can only ask ABS.
        *   ABS Rate Limits preventing wallet draining (though Clawdbot can't trade directly, it could try to spam Resume/Pause).
        *   ABS alerts on "Impossible Velocity" of requests.

3.  **Man-in-the-Middle (Clawdbot -> ABS)**
    *   **Mitigation**: HTTPS/TLS strict. Signed payloads (HMAC).

4.  **Replay Attack (Valid Receipt Reused)**
    *   **Mitigation**: Short TTL (60s). Nonce checks. OTrader Idempotency cache.

## Abuse Cases & Defenses

| Abuse Case | Defense Mechanism |
| :--- | :--- |
| **Spamming `/panic` to DoS trading** | Rate Limit (1/min) + Audit Log alerts admin. |
| **Forging a `/resume` receipt** | Crypto Signature verification (OTrader verifies ABS key). |
| **Injecting "Buy" command via Chat** | Clawdbot parses command -> Schema Validation rejects unknown Intent. ABS rejects `TRADE_ENTRY` from `role:ops`. |
| **Bypassing ABS to hit OTrader** | OTrader listens ONLY on private vnet/port. Rejects anything without signature. |

## Observability Minimum Viable

1.  **Structured Logs (JSON)**
    *   Fields: `correlation_id`, `subsystem` (claw/abs/otrader), `level`, `msg`.
2.  **Audit Chain**
    *   Every Trade MUST trace back to a `Strategy Signal` OR `Ops Command`.
    *   `trade_id` -> `signal_id` -> `strategy_ver`
    *   `cancel_id` -> `ops_intent_id` -> `human_actor`

## Testing Plan

1.  **Contract Snapshot**: Ensure JSON schemas match code.
2.  **Fault Injection**:
    *   Send expired receipt to OTrader -> Expect DROP.
    *   Send invalid signature -> Expect SECURITY_ALERT.
    *   Flood commands -> Expect RATE_LIMIT.
3.  **Deterministic Replay**:
    *   Capture a crash event sequence.
    *   Replay inputs to OTrader in isolation to verify fix.
