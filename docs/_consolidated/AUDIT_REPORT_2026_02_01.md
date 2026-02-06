# Auditoria Técnica Completa — abs-core

## 1. Identificação

- **Projeto**: abs-core
- **Versão**: 2.0.0 (Core) / v0.1.0-alpha (Estado Atual)
- **Domínio**: Governança de Agentes Autônomos (Middleware de IA)
- **Estágio**: Alpha / Validação Arquitetural
- **Criticidade**: Alta (Gatekeeper de decisões de IA)
- **Data da Auditoria**: 2026-02-01
- **Auditor Responsável**: Antigravity (Via Análise Estática)

---

## 2. Objetivo da Auditoria

Avaliar a integridade dos mecanismos de governança ("The Governance Gate"), a imutabilidade dos logs de auditoria e a segurança contra "Excessive Agency" no runtime do ABS.

---

## 3. Contexto Técnico

### Stack
- **Runtime**: Node.js (>=18), Cloudflare Workers (compatível)
- **API Framework**: Hono
- **Dados**: SQLite (`better-sqlite3`) para logs e estado local.
- **Criptografia**: `@noble/hashes`, `node:crypto` (Ed25519 signatures).
- **IA / Automação**: SDK Model Context Protocol (MCP), LLM Agnostic.
- **Validação**: Zod para schemas de eventos e envelopes.

### Restrições
- **Segurança**: Política de "Negar por Padrão" (Deny-by-default).
- **Compliance**: Logs de auditoria imutáveis e assinados criptograficamente.
- **Performance**: Baixa latência exigida no `EventProcessor`.

---

## 4. Análise Arquitetural

### Achados
- **Coesão e acoplamento**: Alta coesão no Core (`processor.ts`, `policy.ts`). Acoplamento fraco via interfaces (`Executor`, `PolicyEngine`).
- **Invariantes explícitas**: 
  - `policy.ts`: Verifica confiança (< 0.8 nega) e whitelist de ações.
  - `events.ts`: Garante hash linkado (blockchain-lite) para imutabilidade.
- **Pontos únicos de falha**: 
  - Banco de dados SQLite local (não escalável horizontalmente sem litestream/d1).
  - `SimplePolicyEngine` hardcoded.
- **Escalabilidade**: Limitada pelo write-lock do SQLite em alta concorrência. Modelo de fila (`EVENTS_QUEUE`) mitiga, mas persistência é gargalo.

### Diagnóstico
- **Arquitetura atual suporta produção?** SIM (Para cargas controladas/Single-Tenant).
- **Arquitetura suporta escala?** NÃO (SQLite local exige migração para D1/Postgres para escala horizontal).

---

## 5. Qualidade de Código

- **Organização estrutural**: Clara separação em `core`, `infra`, `api`, `security`.
- **Contratos e validações**: Uso extensivo de Zod (`EventEnvelopeSchema`) previne dados sujos.
- **Tratamento de erros**: `Executor` captura erros e falha "closed" (retorna `isSuccess: false` sem quebrar o processo).
- **Divida técnica identificada**: 
  - `SimplePolicyEngine` possui regras hardcoded (`ALLOWED_AUTO_ACTIONS`). Isso exige deploy para mudar regras.
  - Inicialização do `AsymmetricSigner` no construtor do `EventProcessor` é um anti-pattern (async no constructor).

---

## 6. Segurança & Confiabilidade

- **Superfície de ataque**: Reduzida. API exibe apenas rotas de eventos autenticadas (`requireScope`).
- **Gestão de segredos**: `.env` usado, mas chaves de assinatura (Ed25519) geradas ephemeramente em `initSigner` (risco de perda de raiz de confiança após restart se não persistidas).
- **Isolamento**: `Sanitizer` (regex) atua antes do LLM.
- **Fail-safe**: 
  - Timeout ou Erro no LLM -> Nenhuma ação executada.
  - Score de Risco >= 80 -> Force DENY.

---

## 7. Observabilidade & Operação

- **Logs**: Estruturados e assinados (`Logger` em `logger.ts`).
- **Rastreabilidade**: `trace_id` propagado desde a entrada (`events.ts`) até a decisão e log.
- **Métricas**: `Metrics.recordDecision` e `breakdown` de latência implementados.

---

## 8. Governança & Decisão

- **Como decisões são tomadas**: `EventProcessor` orquestra: Validação -> Sanitização -> LLM Propose -> Policy Evaluate -> DB Log -> Execute.
- **Rastreabilidade**: Tabela `decision_logs` contém JSON completo + Assinatura HMAC + Link para hash anterior.
- **Auditoria imutável**: Implementada via Hash Chain em `events_store` e `decision_logs`.

---

## 9. Matriz de Riscos

| ID | Categoria | Impacto | Probabilidade | Severidade | Recomendação |
| --- | --- | --- | --- | --- | --- |
| R01 | Escalabilidade | Bloqueio de DB em alta carga | Alta | Crítica | Migrar para Cloudflare D1 ou Postgres remoto para logs. |
| R02 | Segurança | Perda de chaves de assinatura (Efêmeras) | Média | Alta | Persistir chaves Ed25519 em Vault/Secret Manager. |
| R03 | Manutenção | Política Hardcoded (Requer Deploy) | Alta | Média | Implementar `DynamicPolicyEngine` (carregar regras via JSON/DB). |
| R04 | Segurança | Bypass de Sanitizer (Regex frágil) | Média | Alta | Adicionar modelo classificador de Injection (Prompt Shield) além de Regex. |

---

## 10. Conclusão Executiva

- **Nota de maturidade (0–5)**: 3.5 (Core Sólido, Operacionalização Imatura).
- **Apto para produção**: Sim (Escopo restrito/Bêta).
- **Apto para escala**: Não (Gargalo de SQLite).
- **Próximo passo crítico**: Externalizar a configuração de Políticas (sair do hardcoded) e persistir chaves de identidade.

---

## 11. Pergunta Final

**Qual é a versão mais forte deste sistema que ainda não foi considerada?**

Uma versão onde a **Política não é código, é dado vivo**. Onde o `PolicyEngine` roda via WebAssembly (Wasm) baixado dinamicamente de um registro de governança central, permitindo atualização de regras de segurança em tempo real em milhares de agentes sem redeploy do código do agente. Além disso, a substituição do SQLite por um Ledger distribuído (ou Merkle Log transparente) tornaria a auditoria publicamente verificável.
