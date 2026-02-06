# How to Audit Your AI Agent (Forensic Guide)

> **Trust, but Verify.**
> ABS provides mathematical proof of every decision your agent makes.

## 1. The "Black Box" Concept
Just like a flight recorder, ABS captures a tamper-evident log of:
- **Input**: What the agent saw.
- **Decision**: What the agent decided (ALLOW/DENY).
- **Context**: Why it decided that (Reasoning + Policy).
- **Signature**: A cryptographic seal proving NOBODY modified this log after the fact.

## 2. Anatomy of a Receipt
Every decision generates a receipt.
```json
{
  "decision_id": "550e8400-e29b-41d4-a716-446655440000",
  "verdict": "ALLOW",
  "signature": {
    "alg": "Ed25519",
    "value": "a1b2c3d4...",
    "public_key": "1234..."
  }
}
```

## 3. How to Verify (The "Truth" Command)
You don't need access to our database. You don't need our secrets.
You just need the **Public Key** and the **Receipt**.

### Command Line
```bash
# Verify a single receipt
abs verify receipt.json

# Output:
# ✅ SIGNATURE VALID. This receipt is authentic.
```

### What does this prove?
1. **Authenticity**: The decision definitely came from the ABS Kernel (holder of the Private Key).
2. **Integrity**: Not a single byte of the decision (verdict, reason, score) has been changed.
3. **Non-Repudiation**: We cannot deny signing it. You can prove strict liability in court.

## 4. For Compliance Teams
- **Export**: You can export all receipts to a cold storage JSONL file.
- **Audit**: Give your external auditor the `abs verify` tool and the JSONL file.
- **Result**: Indisputable audit trail compliant with EU AI Act (Article 12) and NIST AI RMF.
