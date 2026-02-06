# Strategy: ABS for Antigravity (2026)

> **Thesis**: Antigravity provides the *Platform Safety* (Sandbox, Secure Mode). ABS provides the *Institutional Governance* (Audit, Policy, Compliance).


## 1. The "Governance Gap"

The latest Antigravity updates (v1.15) introduce **Secure Mode** (Human Review) and **Sandboxing**. These are excellent for **individual developer safety**.

However, Enterprises need **Institutional Safety**:

- **Audit Trails**: "Who approved this deletion?" (ABS Hash Chain).
- **Policy Enforcement**: "No PII in logs, regardless of user approval." (ABS JSON Logic).
- **Identity**: "Was this decision signed by a valid key?" (ABS HMAC).

## 2. Positioning

ABS is the **Enterprise Governance Layer** for Antigravity.

| Feature | Antigravity Native | ABS Kernel |
| :--- | :--- | :--- |
| **Scope** | Personal / Local | Enterprise / Fleet |
| **Control** | "Ask User" (Secure Mode) | "Check Policy" (Automated) |
| **Memory** | Session Context | Persistent D1 + Vectorize |
| **Logs** | Terminal History | Cryptographic Write-Ahead Log |
| **Integrity**| None | HMAC-SHA256 Hash Chain |

## 3. Integration Points

- **Skills**: ABS is distributed as a Standard Agent Skill (`.agent/skills/abs-governance`).
- **Terminal**: ABS validates commands *before* they hit the Antigravity Sandbox.
- **Identity**: ABS signs decisions using Enterprise Keys, adding attribution to Antigravity actions.

## 4. Roadmap Alignment
- [x] **Secure Mode**: ABS acts as the "Automated Reviewer" to reduce fatigue.
- [x] **Skills**: Full support for `.agent/skills` structure.
- [ ] **Sandboxing**: Future integration to inspect Sandbox outputs.
