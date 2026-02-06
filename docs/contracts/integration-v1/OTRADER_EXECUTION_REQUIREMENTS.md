# OTrader Execution Requirements (OER)
> **Role**: Executor
> **Trust Model**: Zero Trust (Verify Receipt)

## 1. Receipt Verification Protocol

OTrader **MUST NOT** parse or act upon any command payload directly. It **MUST** extract the `ABS_RECEIPT` envelope.

### Verification Steps
1.  **Existence**: Is `x-abs-receipt` present?
2.  **Signature**: Verify `hmac_sha256(receipt.payload, ABS_PUBLIC_KEY)` matches `receipt.signature`.
3.  **Freshness**: `receipt.timestamp` > `now - 60000ms` (1 min max delay).
4.  **Target**: `receipt.target_system` == `OTRADER_PROD`.

**Code Snippet (Pseudo-Python):**
```python
def on_command(command):
    receipt = command.get('receipt')
    if not abs_verifier.verify(receipt):
        logger.audit("SECURITY_INCIDENT", "Invalid receipt received", origin=command.source)
        return  # SILENT DROP

    # Only now execute logic
    executor.dispatch(receipt.action, receipt.params)
```

## 2. Idempotency & Replay Protection

*   **Idempotency Key**: Every ABS Receipt generates a deterministic `receipt_id`.
*   OTrader MUST maintain a `processed_receipts` cache (TTL 5 mins).
*   If `receipt_id` represents an action already taken, return `SUCCESS` (idempotent) without re-executing side effects via Binance API.

## 3. Local Kill-Switch (Latch)

OTrader must implement a local "Latch" (Circuit Breaker) that persists to disk/db.
*   **State**: `RUNNING` | `PAUSED` | `KILLED`
*   **Behavior**:
    *   If `PAUSED`: Accept only `OPS_RESUME`, `OPS_PANIC`, `OPS_STATUS`. Reject strategy signals.
    *   If `KILLED`: Reject ALL except manual SSH/Console reset.
*   **Trigger**: A valid `OPS_PANIC` receipt switches state to `KILLED`.

## 4. Secret Segregation

*   `BINANCE_API_KEY` / `BINANCE_SECRET_KEY` are injected into OTrader env/vault ONLY.
*   Clawdbot **NEVER** knows these keys.
*   ABS **NEVER** knows these keys.
*   OTrader **NEVER** exposes these keys in logs or responses.

## 5. Heartbeat to ABS

OTrader must send a periodic `HEARTBEAT` (every 30s) to ABS:
*   Payload: `status`, `open_orders_count`, `exposure`, `last_receipt_id`.
*   Purpose: Allows ABS to detect "Zombie Executor" (ABS orders Stop, but Heartbeat shows Running).
