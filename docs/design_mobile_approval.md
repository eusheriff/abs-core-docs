# Mobile Approval Protocol (Human-in-the-Loop)
## "The Nuclear Football"

### 1. Objective
Enable high-risk actions (e.g., transfers >$10k, `rm -rf`, schema drops) to pause execution and require explicit, cryptographic approval from a human administrator via a mobile device/dashboard.

### 2. The Flow
1.  **Trigger**: Agent attempts a sensitive tool call (e.g., `stripe.transfer`).
2.  **Intercept**: Hypervisor detects risk and returns `REQUIRE_HUMAN_SIGNATURE`.
3.  **Hold**:
    *   `mcp-proxy` halts the request.
    *   Generates a `TransactionChallenge`: `SHA256(Action + Params + Nonce)`.
    *   Writes to `pending_approvals.json` (Queue).
4.  **Notify**: Dashboard/Mobile App polls queue and alerts Admin.
5.  **Sign**:
    *   Admin reviews intent ("Transfer $10k to External?").
    *   Admin clicks "APPROVE".
    *   Mobile App signs the `TransactionChallenge` with Admin's Ed25519 Private Key.
6.  **Execute**:
    *   Signature is posted back to the Agent.
    *   Hypervisor verifies signature against `admin_public_key`.
    *   If valid, `ACTION` flips to `ALLOW`.
    *   Agent resumes execution.

### 3. Data Structures

#### Pending Approval
```json
{
  "id": "tx-1234-5678",
  "agent_id": "agent-alpha",
  "tool": "stripe_transfer",
  "params": "{\"amount\": 10000, \"dest\": \"acct_X\"}",
  "timestamp": 1700000000,
  "status": "PENDING"
}
```

#### Admin Approval (Payload)
```json
{
  "tx_id": "tx-1234-5678",
  "decision": "APPROVED",
  "admin_id": "ciso-001",
  "signature": "ed25519_signature_hex"
}
```

### 4. Implementation Strategy (V3.0 Simulation)
- **Queue**: Use a shared file `config/approval-queue.json`.
- **Polling**: Update `Interceptor` to loop/wait while checking the file status.
- **Dashboard**: Add "Pending Approvals" section to `kma-dashboard`.
- **Admin Key**: Generate a mock Admin Keypair for the Dashboard to use.

---
**Security Note**: In production, the "Hold" mechanism would typically involve a callback URL or a Promise that resolves upon WebSocket event, rather than file polling.
