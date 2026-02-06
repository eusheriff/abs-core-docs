# ADR-005: Replacement of @noble/hashes with Node.js Crypto

## Status
Accepted

## Context
During the release preparation for ABS v0.1.0-alpha, we encountered persistent module resolution issues with `@noble/hashes` v2 in the monorepo environment (specifically within `@abs/audit` and `@abs/policy`). The error `Cannot find module '@noble/hashes/sha256'` persisted despite multiple configurations of `tsconfig.json` and attempts to fix import paths (`sha256` vs `sha2`).

This blocked the build of `Clawdbot`, which depends on these packages.

## Decision
We decided to remove the dependency on `@noble/hashes` for hashing operations (SHA-256) in favor of Node.js native `crypto` module (`import { createHash } from 'node:crypto'`).

For Ed25519 signatures, we continue to use `@noble/ed25519` v3, which has proven stable.

## Consequences
- **Pros:** 
  - Eliminates build fragility related to ESM/CJS interop and deep exports resolution of `noble-hashes`.
  - Reduces bundle size slightly (using native platform primitives).
  - Guaranteed stability in Node.js environments (CLI, Bot, Adapter).
- **Cons:**
  - If we port ABS to the browser in the future, we will need to polyfill `node:crypto` or re-introduce a browser-compatible hashing library (or use Web Crypto API).
