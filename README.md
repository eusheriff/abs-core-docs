# ABS Core (`@abs/cli`)
> **The Immune System for AI Agents.**
> A local firewall that intercepts, audits, and blocks risky agent actions before they happen.

![Build Status](https://img.shields.io/badge/build-passing-brightgreen)
![Version](https://img.shields.io/badge/version-1.0.0--beta.1-blue)
![License](https://img.shields.io/badge/license-Apache--2.0-green)

## 🛑 The Problem
Autonomous Agents (LangChain, AutoGPT, CrewAI) are powerful but dangerous. They can:
- `rm -rf` your project.
- Exfiltrate `.env` secrets.
- Open reverse shells.
- Spin up crypto miners.

## 🛡️ The Solution: ABS Core
ABS is a **Runtime Policy Engine** that wraps your agent's execution. It acts as a sudo-like gatekeeper, enforcing regex-based and heuristic policies.

### Features (MVP)
*   **CLI Firewall**: Runs locally, wraps any command (`abs run ...`).
*   **Policy Engine**: Blocks high-risk patterns (`rm -rf`, `nc -e`, `key exfil`) by default.
*   **Gateway Mode**: Exposes a local HTTP server (`POST /audit`) for agents to query permission.
*   **Zero Latency**: Runs entirely on your machine. No cloud dependency.

## 🚀 Quick Start

### 1. Installation
```bash
npm install -g @abs/cli
```

### 2. Usage (CLI Wrapper)
Wrap any dangerous command to protect it:
```bash
# This will be BLOCKED
abs run "rm -rf /"

# This will be WARNED (Secret access)
abs run "cat .env"

# This is ALLOWED
abs run "ls -la"
```

### 3. Usage (Gateway Mode)
For Python/Node agents (LangChain, etc.):
```bash
# Start the Policy Server
abs serve -p 3000
```

Then query it from your agent:
```python
import requests

def ask_abs(command):
    res = requests.post("http://localhost:3000/audit", json={"command": command})
    if res.status_code == 403:
        raise PermissionError("ABS Blocked this action")
    return True
```

## 🛠️ Roadmap (The Reality)
- [x] CLI Scaffolding
- [x] Policy Engine (Regex)
- [x] Local Gateway (Fastify)
- [ ] User Dashboard (Cloud)
- [ ] Hardware TEE Integration (Nitrol Enclaves)

## 📄 License
Apache 2.0 - Open Source.
Created by OConnector.
