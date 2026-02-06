# AUDITORIA TÉCNICA FORENSE (ABS V6.5)

**AUDITOR**: Antigravity (Code-First Legist)
**METODOLOGIA**: Evidência Verificável (Code is Law)
**DATA**: 2026-02-05

---

## 0. RESUMO EXECUTIVO (REALIDADE OBSERVADA)

O ABS V6.5 não é um "Sistema Operacional para IA".
**OBSERVADO**: É um **Middleware de Interceptação de IO (Input/Output)** para servidores MCP (Model Context Protocol).
**FUNÇÃO**: Ele intercepta fluxos `STDIN/STDOUT` entre um Agente LLM e suas Ferramentas, aplicando regras de bloqueio antes que a ferramenta receba o comando.
**VALOR CONCRETO**: Impede execução de ferramentas não autorizadas via "Active Blocking".
**RISCO ESTRUTURAL**: A arquitetura depende de piping de processos Node.js (`spawn`), o que introduz latência e fragilidade se o processo pai morrer.

---

## 1. MAPA DO REPOSITÓRIO (ESTRUTURA REAL)

| Pasta | Responsabilidade (Código) | Status |
| :--- | :--- | :--- |
| `packages/mcp-proxy` | **O Sentinel**. Entrypoint que inicia o processo filho e analisa JSON-RPC. | ✅ **DRIVER** |
| `packages/core` | **O Runtime**. Contém CLI (`verify`), Lógica de ROI e Gravador. | ✅ **TOOLING** |
| `packages/hypervisor-wasm` | **O Cérebro**. Módulo Rust compilado para WASM que contém as regras de regex. | ✅ **KERNEL** |
| `packages/policy` | **As Regras**. Definições estáticas (JSON/YAML) de permissões. | ✅ **CONFIG** |
| `packages/hypervisor` | **Código Morto**. Versão legada em JS. | 💀 **ZOMBIE** |

---

## 2. ENTRYPOINTS E FLUXO (HAPPY PATH)

### Modo de Execução: "Sentinel Wrap"
**OBSERVADO**: O sistema roda envolvendo o comando original.
`abs-sentinel --target "npx my-tool-server"`

### The Happy Path (Fluxo de Vida de 1 Request)
1.  **Agente** envia JSON-RPC `tools/call` via STDIN.
2.  **Sentinel** (`mcp-proxy/src/index.ts:31`) captura o chunk de texto.
3.  **Interceptor** (`interceptor.ts`) analisa o payload:
    *   Valida Assinatura (Se aplicável).
    *   Verifica Circuit Breaker.
    *   Chama `hypervisor.validateIntent` (WASM).
4.  **Decisão**:
    *   Se `ALLOW`: Sentinel escreve no STDIN do processo filho (`child.stdin.write`).
    *   Se `DENY`: Sentinel **não escreve** e retorna erro sintético JSON-RPC para o Agente (`process.stdout.write`).
5.  **Child Process** executa a ferramenta e retorna resultado.
6.  **Sentinel** captura STDOUT do filho, atualiza métricas de sucesso e repassa para o Agente.

---

## 3. MODELO DE DADOS & SEGURANÇA

### Entidades & Invariantes
*   **Decisão (Verdict)**: Deve ser binária (`ALLOW` / `DENY`).
*   **Audit Log**: Hash Chain (`prev_hash` linkado).
    *   **OBSERVADO**: `ABSAuditSystem` (Rust) usa `Sha256` + `Ed25519`.

### Estratégia de Erros
*   **Fail-Closed**: O Bug de parse error foi corrigido.
    *   *Status*: ✅ **FIXED**. Parse errors agora derrubam o pacote (Drop) em vez de encaminhar. Código auditado em `mcp-proxy/index.ts`.

### Segurança Real
*   **AuthN**: Baseada em chaves Ed25519 (CLI).
*   **Isolamento**: Não existe isolamento de memória real. O Sentinel roda no mesmo user-space que o Agente ou a Ferramenta (dependendo do deploy).
*   **Secrets**: Lidos de `process.env`. Se o atacante tiver RCE (Remote Code Execution) no container, ele lê as chaves.

---

## 4. VEREDITO FINAL

O ABS é um **Firewall de STDIN para Protocolo MCP**.
Ele protege ferramentas críticas (banco de dados, API de pagamentos) de serem acionadas por comandos maliciosos ou alucinados de um LLM. Funciona bem para impedir "acidentes", mas não é um Hypervisor de nível militar (ainda).

### 🔥 A Versão Mais Forte (Forensic Pivot)

**"Qual é a versão mais forte deste entendimento?"**

O **"Native Kernel Module"**.
Atualmente, o ABS roda em **User Space** (Node.js). Isso é lento e hackeável.
A versão definitiva deve rodar como um **eBPF (Extended Berkeley Packet Filter)** no Kernel do Linux.
*   **Por que?**: O eBPF intercepta chamadas de sistema (syscalls) antes mesmo do Node.js saber que elas existem.
*   **Vantagem**: Performance zero-copy e impossibilidade de bypass pelo usuário (rootkit-proof). O ABS deixa de ser "Middleware" e vira "Parte do OS".

---
**Auditoria Forense Concluída.**
*Antigravity Labs*
