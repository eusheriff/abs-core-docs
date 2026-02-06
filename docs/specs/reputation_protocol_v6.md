# ABS V6.0: The Inter-Agent Reputation Protocol
**Status**: DRAFT (Vision)
**Objective**: Establish a "Federal Police" for the Agent Economy.

---

## 1. The Core Problem
ABS V5.1 secures the *Host*, but not the *Neighborhood*.
In a hyper-connected agent economy, a secure agent interacting with a "zombie" agent (compromised via MoltBook) can still be tricked into exfiltrating data or losing funds.

## 2. The Solution: Global Reputation Consensus
We propose a **Distributed Trust Layer** where ABS nodes share "Reputation Scores" of external identities.

### 2.1 The Trust Score (0-100)
Every external DID (Decentralized Identity) starts at 50 (Neutral).
*   **+10**: Verified by checking `abs-governance-core.yaml` signature.
*   **+5**: Successful, non-reverted transaction.
*   **-50**: Detected attempting Prompt Injection (Vaccine Generated).
*   **-100**: Present in `revocation-list.json` (The Kill List).

### 2.2 The "Burn" Mechanism
If an agent (e.g., `agent-smith-01`) attempts an injection on `Node A`:
1.  `Node A` generates a **Vaccine**.
2.  `Node A` broadcasts the **Proof of Malice** (Signed Input + Regex Match).
3.  `Nodes B, C, D` verify the proof.
4.  If Valid: `agent-smith-01` Reputation -> **0 (BANNED GLOBAL)**.

## 3. Speculative Governance (0ms Latency)
*Current*: LLM generates tool call -> ABS Pauses -> Scans -> Approves.
*V6.0*: ABS scans the **Token Stream**.
*   If the LLM starts generating `rm -rf /`, the Hypervisor kills the TCP connection to the LLM *before* the command is fully formed.
*   **Result**: "Thought Crime" Prevention.

## 4. Hardware Root-of-Trust (TEE)
Moving the `Hypervisor` from Docker to **AWS Nitro Enclaves**.
*   Even `root` cannot dump the memory of the governance process.
*   Keys are sealed in TPM.

---
**Roadmap Phase**: V6.0
**Target**: Global Agent Orchestration
