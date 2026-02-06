# ADR-001: Estratégia Open Core para ABS

**Status**: Aceito  
**Data**: 2026-01-19  
**Decisor**: Rodrigo Gomes (OConnector Technology)

## Contexto

O ABS Core é uma fundação para sistemas de negócio autônomos. A decisão sobre modelo de licenciamento afeta:
- Adoção pelo mercado
- Captura de valor comercial
- Confiança e transparência

## Decisão

Adotar modelo **Open Core** com:
- **Core aberto (Apache-2.0)**: Protocolos (MCP/HTTP), interfaces, specs, state machine genérica, e o mecanismo básico de intercepção (Sentinel CLI).
- **Componentes fechados (Commercial)**: Dashboards de segurança e econômicos, APIs de logs persistentes, conectores enterprise, e automação de incidentes.

## Justificativa

1. **Monetização**: Move o "valor visual" e a "memória operacional" para o Enterprise, mantendo a "proteção" acessível mas básica no Core.
2. **Separação de Preocupações**: O Core garante que o robô não faça besteira; o Enterprise mostra *por que* ele não fez e gera relatórios de auditoria.

## Componentes Abertos

- Event Envelope e Schemas (MCP/JSON-RPC)
- Sentinel Proxy (CLI básico para intercepção)
- Interface de Human-in-the-loop (via terminal/stdout)
- Exemplos de integração básica

## Componentes Fechados

- **Dashboard de Segurança (UI/Grafana/NextJS)**
- **API de Logs de Decisão e Auditoria Retrospectiva**
- Métricas econômicas, SLAs e Performance
- Policy packs avançados e conectores enterprise
- Console de Gestão de Identidade e Permissões

## Consequências

- **Positivas**: Adoção facilitada, confiança pública, thought leadership
- **Negativas**: Concorrentes podem usar core aberto sem pagar
- **Mitigação**: Valor real está na operação (policies + connectors + suporte)
