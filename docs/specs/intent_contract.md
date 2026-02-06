# ABS V3.0: The Intent Contract (JSON)

## 1. The Intent Object
This payload travels from the Hypervisor to the Admin's Mobile Device. It explicitly binds the "Risk" to the "Approver".

```json
{
  "abs_version": "2.0.0",
  "intent_id": "tx-8892-af21",
  "timestamp": 1738778400,
  "agent": {
    "id": "manu-abs-01",
    "risk_score": 85
  },
  "action": {
    "tool": "cloud_provider/scale_infrastructure",
    "params": {
      "instance_type": "p4d.24xlarge",
      "count": 10,
      "region": "us-east-1"
    },
    "estimated_cost": "$32.400/mo",
    "rationale": "Deteção de pico de tráfego anómalo no módulo de pagamentos."
  },
  "security_gate": {
    "required_sig": "CISO_ADMIN_KEY",
    "nonce": "a7b2...f310",
    "risk_score": 85,
    "timestamp": 1707158000,
    "policy_version": "v3.1.0",
    "rationale": {
        "code": "EMERGENCY_FIX",
        "text": "Correcting production outage (INC-992)",
        "selected_by": "human_operator"
    }
  }
}
```

## 2. The Verification Logic (Rust)
The Hypervisor performs the following check:

1.  **Hash**: `SHA256(Canonical(IntentObject))`
2.  **Verify**: `Ed25519_Verify(Signature, Hash, AdminPublicKey)`

This ensures **Non-Repudiation** (The admin cannot deny signing *exactly* this intent).

### 2.2 Rationale Codes (ARL)
The `rationale` block is MANDATORY for all high-risk signatures (>70).
*   `EMERGENCY_FIX`: Bypassing standard protocol for immediate remediation.
*   `PLANNED_MAINTENANCE`: Pre-approved change window operations.
*   `FALSE_POSITIVE`: Overriding a designated false threat detection.
*   `DRILL`: Simulated attack handling.

### 2.3 The Binding (What is signed?)
The signature validates the hash of the *entire* canonical JSON, including the selected rationale.
`Sign( Hash( intent_json + rationale_json ) )`
