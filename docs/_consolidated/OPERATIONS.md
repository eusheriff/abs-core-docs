# ABS-Neuro Operations & Deployment Guide

## 1. System Requirements
ABS-Neuro utilizes advanced machine learning capabilities via `ONNX Runtime` and `Transformers.js`. These libraries rely on native binaries that must be compiled or downloaded for the specific target architecture (e.g., Linux x64, macOS ARM64).

### Prerequisites
- **Node.js**: v18+ (LTS recommended)
- **Compilers**: `g++`, `make`, `python3` (for `node-gyp` builds of `sharp` and `better-sqlite3`).
- **Memory**: Minimum 2GB RAM (4GB recommended) for Model Inference.

## 2. Installation (Production)

When deploying to a Production Environment (Linux Server / Docker), you **MUST** ensure native modules are rebuilt.

```bash
# 1. Install Dependencies
pnpm install --prod

# 2. Rebuild Native Modules (CRITICAL)
# This fixes "sharp" and "better-sqlite3" missing binary errors
pnpm rebuild sharp better-sqlite3 onnxruntime-node
```

## 3. Model Provisioning (ABS-Neuro)

The `NeuroEngine` attempts to download the embedding/classification models securely from HuggingFace on first run.

- **Firewall Rule**: Allow HTTPS outbound to `huggingface.co`.
- **Cache**: Models are stored in `packages/neuro/models`. Ensure this directory is writable.

### Pre-Baking Models (Docker)
To avoid runtime downloads, you can pre-download models during the build phase:

```bash
cd packages/neuro
npm run download-model
```

*Note: This step requires `sharp` to be working in the build environment.*

## 4. Dockerfile Example (Ubuntu/Debian)

```dockerfile
FROM node:20-bullseye

# Install Build Tools for Native Modules
RUN apt-get update && apt-get install -y python3 make g++

WORKDIR /app
COPY . .

# Install & Rebuild
RUN npm install -g pnpm
RUN pnpm install --frozen-lockfile
RUN pnpm rebuild sharp better-sqlite3 onnxruntime-node

# Pre-download Models (Optional)
# RUN cd packages/neuro && npm run download-model

# Start
CMD ["pnpm", "start"]
```

## 5. Troubleshooting

### "Sharp Missing" / "GLIBC Error"
- **Cause**: The binary downloaded matches your Dev OS (Mac) but you are running on Prod (Linux), or `pnpm` ignored the build script.
- **Fix**: Run `pnpm rebuild sharp` explicitly on the target machine.

### "Model Load Failed"
- **Behavior**: ABS-Neuro enters **Fail-Safe Mode**. It will log warnings (`⚠️ Neuro Warning`) but will **NOT** block traffic. It degrades to using the Vector Store with Mock Embeddings (if configured) or just allows traffic (if policy permits).

## 6. Enterprise Key Management & Rotation

### Secret Key Requirements
To run ABS in production (`NODE_ENV=production`), you must provide a cryptographically strong `ABS_SECRET_KEY`.
- **Length**: Minimum 32 characters.
- **Complexity**: Random bytes (e.g., generated via `openssl rand -hex 32`).

### Rotation Procedure
ABS supports zero-downtime key rotation via **Dual-Key Validation** (Planned Feature). Currently, rotation requires a rolling restart:

1.  **Generate New Key**:
    ```bash
    openssl rand -hex 32
    ```
2.  **Update Secrets Manager**:
    - Update `ABS_SECRET_KEY` in Cloudflare Dashboard or Vault.
3.  **Redeploy**:
    - Trigger a new deployment. The `validateSecrets()` hook will verify strength on boot.

