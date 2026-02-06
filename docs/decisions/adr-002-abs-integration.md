# ADR-002: Integração do ABS (Agent Behavior System) no Clawdbot

## Status
Aceito

## Contexto
O Clawdbot (Moltbot) necessita de uma camada de governança determinística para garantir operações seguras e auditáveis de seus agentes AI, especialmente em ambientes financeiros e de alta sensibilidade. O framework ABS fornece essa governança.

## Decisão
Integrar o ABS diretamente no núcleo do Clawdbot via:
1.  **SDK Local (`src/abs/`)**: Módulo dedicado contendo o cliente ABS, adaptador de políticas e lógica de avaliação.
2.  **Hook de Política (`pi-tools.policy.ts`)**: Interceptação síncrona na função `isToolAllowedByPolicies` para aplicar regras ABS (verdict ALLOW/DENY).
3.  **Configuração Tipada**: Extensão do `MoltbotConfig` para incluir configurações ABS (`enabled`, `mode`, `policies`).
4.  **Auditabilidade**: Geração de `chainHash` e `evidence` para cada decisão tomada.
5.  **Skill Dedicada**: Disponibilização da governança como skill (`abs-governance`) para conhecimento do agente.

## Consequências
### Positivas
-   **Segurança Determinística**: Kill-switch (`SAFE_MODE`) e limites rígidos (R0-R3) aplicados no nível do código.
-   **Auditoria**: Rastreabilidade de decisões através de hash chain e logs.
-   **Flexibilidade**: Suporte a políticas locais e futuras remotas.

### Negativas
-   **Dependência**: O core do Clawdbot agora depende do módulo ABS.
-   **Performance**: Leve overhead na verificação de cada tool (mitigado por check síncrono).

## Implementação
-   Módulo: `src/abs/`
-   Config: `src/config/types.abs.ts`
