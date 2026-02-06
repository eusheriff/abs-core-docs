# Auditoria Técnica e Comercial — ABS Kernel (abs-core v2.7.0)

**Data**: 2026-02-01  
**Auditor**: Antigravity (Principal Engineer / M&A Tech Due Diligence)  
**Objeto**: [abscore.app](https://abscore.app) / [github.com/eusheriff/abs-core](https://github.com/eusheriff/abs-core)

---

## 1. Executive Summary

O **ABS Kernel** posiciona-se como um "Sistema Imunológico para Agentes de IA", prometendo governança determinística e auditável para prevenir "Excessive Agency".

*   **O que Funciona**: A suite de testes (`fault-injection`, `invariants`) é surpreendentemente madura, demonstrando uma cultura de engenharia rigorosa e clara compreensão dos riscos de IA.
*   **O que Quebra**: A implementação de segurança é "teatral". As chaves de assinatura são geradas em memória (perdem-se no restart), e o runtime mistura dependências nativas de Node.js (`better-sqlite3`) com promessas de Edge (`Cloudflare Workers`), tornando o deploy em Edge impossível no estado atual.
*   **Veredito**: **Não Apto para Produção**. É um ativo de arquitetura (PoC avançada), não um produto comercializável.
*   **Buyability**: **Baixa**. O valor está na propriedade intelectual dos Testes e na Arquitetura de Policies, não no Código Runtime.

---

## 2. System Map & TCB

```mermaid
flowchart TD
    subgraph "Trusted Computing Base (TCB)"
        API[Hono Gateway]
        WAL[(Event Store - SQLite/D1)]
        PDP[Policy Engine]
        Signer[Ed25519 Signer]
    end
    
    User[AI Agent / Webhook] -->|HTTP POST| API
    API -->|1. Persist| WAL
    WAL -->|2. Introspect| Sanitizer[Prompt Sanitizer]
    Sanitizer -->|3. Propose| LLM[LLM Adapter]
    LLM -->|4. Evaluate| PDP
    PDP -->|5. Sign| Signer
    Signer -->|6. Log| WAL
    WAL -->|7. Execute (Soft Check)| Executor[Action Executor]
```

**TCB Analysis**:
*   **Soft Boundary**: O `Executor` e o `Policy Engine` rodam no mesmo processo. Não há isolamento criptográfico entre a decisão e a execução.
*   **Escape Hatch**: Qualquer desenvolvedor pode instanciar `WebhookExecutor` diretamente, ignorando o PDP.

---

## 3. Threat Model (OWASP LLM Top 10)

| Risco OWASP | Vetor no ABS | Controles Atuais | Gap Crítico |
| :--- | :--- | :--- | :--- |
| **LLM01: Prompt Injection** | Payload malicioso no evento | Regex Sanitizer (`sanitizer.ts`) | Regex é trivialmente bypassável. Falta análise semântica. |
| **LLM08: Excessive Agency** | LLM sugere ação destrutiva | Whitelist no PDP (`policy.ts`) | A policy é hardcoded no código. Requer redeploy para patch. |
| **LLM02: Insecure Output** | Logs contendo PII/Secrets | Nenhum redaction visível | Logs são gravados "raw" no SQLite local. |
| **LLM10: Supply Chain** | Dependência maliciosa | Lockfiles presentes | `better-sqlite3` (C++) impede verificação em runtimes isolados (Edge). |

---

## 4. Crypto Audit (Integridade vs Não-Repúdio)

*   **Integridade Interna**: **OK**. O sistema usa Hash Chain (`integrity.ts`), garantindo que logs não sejam alterados *enquanto o banco de dados persistir*.
*   **Não-Repúdio**: **FALHA CRÍTICA**.
    *   **Evidência**: `src/core/processor.ts:71` -> `AsymmetricSigner.generate(...)`.
    *   **Impacto**: A cada restart do servidor, uma nova chave privada é gerada. Logs assinados há 5 minutos tornam-se inverificáveis matematicamente se o servidor reiniciar. Não há PKI (Infraestrutura de Chaves Públicas).
    *   **NIST SP 800-57**: Violação básica de gerenciamento de chaves (Key Management).

---

## 5. Enforcement Reality Check

*   **Proof of Decision**: **INEXISTENTE**. O `Executor` recebe um objeto JSON simples (`DecisionProposal`). Ele confia cegamente que o objeto veio do PDP.
*   **Inescapabilidade**: **NULA**. O sistema é uma biblioteca (middleware), não um Gateway de Infraestrutura. Se o desenvolvedor não chamar `policy.evaluate()`, nada acontece.

### Classificação:
( ) Enforcement Real (Hard PEP)
(x) Middleware Útil (Soft PEP)

---

## 6. Storage & Scalability

*   **Locking**: Uso de `better-sqlite3` (síncrono) é excelente para latência em *single-tenant*, mas desastroso para concorrência.
*   **Edge Incompatibility**: O `package.json` lista `better-sqlite3` como dependência de produção. Isso **quebra** o deploy em Cloudflare Workers (que não suportam bindings C++). A promessa de "Low Latency Architecture on Edge" do site é, tecnicamente, falsa na versão atual (`v2.7.0`).

---

## 7. Testability & Conformance

*   **Ponto Alto**: A pasta `packages/core/test` contém suites de *Fault Injection* e *Invariants*. Isso vale mais que o próprio código de runtime.
*   **Coverage**: Testes cobrem bem o "Golden Path", mas a falta de integração real com D1 (que é mockado pelos testes locais) esconde o risco de deploy.

---

## 8. Risk Register

| ID | Cat. | Risco | Severidade | Evidência | Correção |
|:---|:---|:---|:---|:---|:---|
| **R01** | Crypto | Chaves efêmeras em memória (Perda de Identidade) | **P0 (BLOCKER)** | `processor.ts:71` | Persistir chaves em Vault/Env Var. |
| **R02** | Arch | Dep C++ (`better-sqlite3`) impede Edge Deploy | **P0 (BLOCKER)** | `package.json:33` | Dual-build: `sqlite` (Node) vs `d1` (Edge). |
| **R03** | Sec | Sanitizer via Regex frágil | **P1 (High)** | `sanitizer.ts` | Integrar modelo classificador (Prompt Shield). |
| **R04** | Ops | Policy Hardcoded requer redeploy | **P2 (Med)** | `policy.ts` | Externalizar regras para JSON/DB. |

---

## 9. Roadmap 3–6 meses

### P0: Foundation (Mês 1)
1.  **Identity Persistence**: Implementar carregamento de chaves via ENV (`ABS_PRIVATE_KEY`).
2.  **Platform Split**: Separar o pacote em `@abs/node` e `@abs/edge` para resolver o conflito do SQLite.

### P1: Security Hardening (Mês 2-3)
1.  **Signed Execution**: Refatorar `Executor` para exigir assinatura válida no envelope de decisão.
2.  **Async Queue**: Remover processamento da thread HTTP principal.

### P2: Enterprise Features (Mês 4-6)
1.  **Remote Policy**: Carregamento dinâmico de políticas (Wasm ou JSON).
2.  **Audit Dashboard**: UI para visualização de logs assinados.

---

## 10. Buyability & Valuation

### Cenário 1: Aquisição de Produto (SaaS)
*   **Veredito**: **Não comprável**. O produto não para em pé (falha de deploy em Edge, falha de auditoria em restart).
*   **Risco**: Alto churn técnico e refatoração completa necessária.

### Cenário 2: Acqui-hire (Equipe/IP)
*   **Veredito**: **Altamente Atrativo**.
*   **Por quê?**: A suite de testes (`fault-injection`) e a arquitetura de conceitos (Invariantes, Hash Chain) demonstram uma equipe que entende profundamente o problema de Governança de IA, mesmo que a execução atual seja imatura.
*   **Valuation**: Custo de Engenharia (Build Cost) + Premium por Expertise (IP de Testes).

### Conclusão para Due Diligence
Recomenda-se a aquisição da **Propriedade Intelectual (Codebase de Testes e Arquitetura)** e da **Equipe**, descontinuando o runtime atual em favor de uma reescrita (v3.0) baseada nos princípios validados pelos testes existentes.
