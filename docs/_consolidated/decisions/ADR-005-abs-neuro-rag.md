# ADR-005: ABS-Neuro Hybrid RAG Architecture

## Status
Accepted (Implemented in P4.5 Prototype)

## Context
The ABS Kernel requires a "Semantic Firewall" to prevent "Confused Deputy" attacks where authorized tools (like `curl` or `filesystem_read`) are used for malicious purposes (data exfiltration, credential harvesting). A purely syntactic firewall (Allow/Deny by tool name) is insufficient.

We initially proposed using pure ONNX-based Small Language Models (SLMs) like `MobileBERT` for Zero-Shot Classification. However, relying solely on on-device ML inference presents challenges:
1. **Latency**: Inference can take >100ms on non-accelerated hardware.
2. **Dependencies**: Libraries like `onnxruntime-node` and `sharp` (for transformers) require native binaries, which may fail in restrictive environments.
3. **Determinism**: ML models are probabilistic; governance often requires deterministic overrides.

## Decision
We have adopted a **Hybrid RAG (Retrieval-Augmented Governance)** architecture for the `NeuroEngine`.

### Layered Defense Strategy
1.  **Layer 1: Static Policy (Existing)**
    - Standard Allow/Deny by Tool Name/User Role.
    
2.  **Layer 2: Cognitive Cache (Vector RAG) - *New in P4.5***
    - **Mechanism**: Compute embedding of the tool payload. Perform Cosine Similarity search against a local Vector Store of known malicious/benign patterns.
    - **Threshold**: If similarity score > 0.9 (configurable), apply the stored label immediately.
    - **Benefit**: <10ms latency, deterministic overrides, easy to "patch" via database updates (Learning Mode).
    
3.  **Layer 3: Zero-Shot Inference (SLM) - *New in P4***
    - **Mechanism**: If no vector match found, run ONNX model (`MobileBERT`) to classify intent dynamically.
    - **Fallback**: If the model fails to load (missing binaries), the system degrades gracefully to Layer 2 only.

## Consequences
- **Positive**: 
    - Resilience: System functions even if ML environment is broken (as proven in Prototype tests).
    - Speed: Common attacks are blocked instantly via Vector Cache.
    - Extensibility: New threats can be added to the Vector DB without retraining models.
- **Negative**:
    - Complexity: Introduces Vector Store management (currently in-memory `SimpleVectorStore`, needs migration to Qdrant/Chroma for production).
    - Storage: Requires maintaining a database of threat embeddings.

## Technical Implementation
- **Components**: `NeuroEngine` (Orchestrator), `SimpleVectorStore` (Cosine Search), `transformers.js` (Embeddings/Inference).
- **Location**: `packages/neuro`.
