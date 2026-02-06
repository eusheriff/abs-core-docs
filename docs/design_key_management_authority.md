# ABS Key Management Authority (KMA) - Architecture Design
## "The CISO Command Center"

### 1. Problem Statement: The "Zombie Agent" Risk
In a distributed agentic system (V3.0), if an agent's private key is compromised or the agent goes rogue (logic loop/hallucination), we need a **Kill Switch**.
- **Current State**: `hypervisor-wasm` blocks specific patterns (Vaccine).
- **Gap**: We cannot revoke the *Identity* of an agent globally without updating every node manually.

### 2. Solution: The KMA (Key Management Authority)
A centralized (or federated) authority that publishes a **Signed Revocation List (SRL)**.
- **Analogy**: SSL Certificate Authority (CA) + CRL (Certificate Revocation List).
- **Mechanism**: The Hypervisor subscribes to the KMA feed. If an Identity is in the SRL, the Hypervisor **refuses to sign** any action, effectively putting the agent in a coma.

### 3. Architecture Components

#### A. The Vault (Secure Storage)
- **Role**: Stores Root Keys (Master Identity).
- **Tech**: HashiCorp Vault / AWS KMS / Cloudflare Workers KV (Encrypted).
- **Action**: Generates Agent Keypairs (Ed25519) and signs their "Birth Certificate".

#### B. The Broadcaster (Pub/Sub)
- **Role**: Pushes updates to the ABS Network.
- **Channel**: `wss://kma.abs-security.io/v1/sync`
- **Payload**: `SignedCommand { action: "REVOKE", target_id: "agent-007", reason: "Compromised" }`

#### C. The Dashboard (UI)
- **User**: CISO / Security Engineer.
- **Capabilities**:
    1.  **Live Grid**: See all active agents and their current Risk Score.
    2.  **Audit Stream**: Real-time feed of the Immutable Hash-Chain events.
    3.  **Nuclear Button**: "Revoke All Keys" or "Revoke Agent X".
    4.  **Key Rotation**: Trigger automated key rotation for the fleet.

### 4. Integration with Hypervisor
The `ABSHypervisor` (Rust) will be updated to check the **Validity Status** of its own identity before signing anything.

```rust
// logical flow in hypervisor-wasm
fn sign_action(&self, payload) -> Result<Signature, Error> {
    if self.kma_client.is_revoked(self.identity_id) {
        return Err("IDENTITY_REVOKED_BY_AUTHORITY");
    }
    // ... proceed to sign
}
```

### 5. Implementation Stages
1.  **Mock KMA**: Simple JSON endpoint serving the "Allow List".
2.  **Dashboard UI**: Next.js App connecting to the Mock KMA and Hash-Chain logs.
3.  **Protocol**: Integrate Client Logic into `mcp-proxy`.

### 6. Value Proposition (IP)
> "We don't just build agents; we give you the keys to turn them off."
