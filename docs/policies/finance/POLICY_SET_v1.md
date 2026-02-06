# ABS Finance Policy Set v1 — Governança Determinística para Trade Automatizado

> **Versão**: 1.0.0  
> **Escopo**: Cripto perpétuos (BTC/ETH) via Binance  
> **Modo**: Determinístico (sem ML/probabilístico no ABS)  
> **Princípio**: Gov ≠ Exec (ABS governa, script executa)

---

## 1. Policy Levels (R0–R3)

Os níveis de risco definem o "regime operacional" do ABS. Cada nível tem **critérios de entrada/saída**, **ações permitidas/bloqueadas** e **limites obrigatórios**.

### R0 — Normal

| Aspecto | Especificação |
|---------|---------------|
| **Descrição** | Condições normais de mercado e infraestrutura |
| **Critérios de Entrada** | TODOS os sinais abaixo dentro dos limites |
| **Critérios de Saída** | Qualquer critério R1+ atingido |

**Condições R0 (todos devem ser TRUE):**
- `ATR_14 < ATR_baseline * 2.0` (volatilidade normal)
- `spread_bps < 10` (spread < 0.10%)
- `ws_latency_ms < 200` (latência WebSocket normal)
- `api_rate_limit_remaining > 50%`
- `daily_drawdown_pct < 3%`
- `weekly_drawdown_pct < 7%`
- `consecutive_losses < 4`
- `exchange_status = "OPERATIONAL"`

**Limites R0:**

| Parâmetro | Limite | Justificativa |
|-----------|--------|---------------|
| `max_position_pct` | 10% do capital | Limita exposição por ativo |
| `max_leverage` | 5x | Alavancagem conservadora |
| `max_slippage_bps` | 15 | 0.15% slippage máximo |
| `max_trades_per_day` | 50 | Evita overtrading |
| `max_orders_cancelled` | 20 | Evita spam de ordens |

**Ações Permitidas:**
- ALLOW: Entradas, saídas, ajustes de posição
- Modo completo de operação

---

### R1 — Cautela

| Aspecto | Especificação |
|---------|---------------|
| **Descrição** | Condições moderadamente adversas |
| **Critérios de Entrada** | QUALQUER condição abaixo TRUE |
| **Critérios de Saída** | NENHUMA condição R1 + TODOS R0 TRUE por 5 min |

**Condições de Entrada R1 (qualquer uma):**
- `ATR_14 >= ATR_baseline * 2.0 AND ATR_14 < ATR_baseline * 4.0`
- `spread_bps >= 10 AND spread_bps < 30`
- `ws_latency_ms >= 200 AND ws_latency_ms < 500`
- `api_rate_limit_remaining <= 50% AND > 20%`
- `daily_drawdown_pct >= 3% AND < 5%`
- `consecutive_losses >= 4 AND < 6`
- `news_severity = "MEDIUM" AND news_confidence >= 0.7`

**Limites R1:**

| Parâmetro | Limite | Delta vs R0 |
|-----------|--------|-------------|
| `max_position_pct` | 5% | -50% |
| `max_leverage` | 3x | -40% |
| `max_slippage_bps` | 10 | -33% |
| `max_trades_per_day` | 25 | -50% |
| `require_confirmation` | TRUE para > 2% capital | Novo |

**Ações:**
- ALLOW: Saídas (sempre)
- RESTRICT: Entradas (tamanho reduzido)
- REQUIRE_CONFIRMATION: Entradas > 2% do capital

---

### R2 — Restrição Forte

| Aspecto | Especificação |
|---------|---------------|
| **Descrição** | Condições adversas significativas |
| **Critérios de Entrada** | QUALQUER condição abaixo TRUE |
| **Critérios de Saída** | NENHUMA condição R2+ por 15 min + R1 ou menor |

**Condições de Entrada R2 (qualquer uma):**
- `ATR_14 >= ATR_baseline * 4.0 AND ATR_14 < ATR_baseline * 8.0`
- `spread_bps >= 30 AND spread_bps < 100`
- `ws_latency_ms >= 500 AND ws_latency_ms < 2000`
- `api_rate_limit_remaining <= 20% AND > 5%`
- `daily_drawdown_pct >= 5% AND < 8%`
- `weekly_drawdown_pct >= 7% AND < 12%`
- `consecutive_losses >= 6 AND < 8`
- `news_severity = "HIGH" AND news_confidence >= 0.6`
- `exchange_status = "DEGRADED"`

**Limites R2:**

| Parâmetro | Limite | Delta vs R1 |
|-----------|--------|-------------|
| `max_position_pct` | 2% | -60% |
| `max_leverage` | 2x | -33% |
| `max_slippage_bps` | 5 | -50% |
| `max_trades_per_day` | 10 | -60% |
| `pause_entries_sec` | 300 | Novo: 5 min entre entradas |

**Ações:**
- ALLOW: Saídas de emergência (sempre)
- RESTRICT: Reduções de posição apenas
- DENY: Novas entradas (exceto confirmação humana)
- REQUIRE_CONFIRMATION: Qualquer operação não-saída

---

### R3 — Emergência (Kill-Switch)

| Aspecto | Especificação |
|---------|---------------|
| **Descrição** | Condições críticas — proteção de capital |
| **Critérios de Entrada** | QUALQUER condição abaixo TRUE |
| **Critérios de Saída** | CONFIRMAÇÃO HUMANA OBRIGATÓRIA |

**Condições de Entrada R3 (qualquer uma):**
- `ATR_14 >= ATR_baseline * 8.0` (volatilidade extrema)
- `spread_bps >= 100` (spread >= 1%)
- `ws_latency_ms >= 2000 OR ws_disconnected = TRUE`
- `api_rate_limit_remaining <= 5%`
- `daily_drawdown_pct >= 8%`
- `weekly_drawdown_pct >= 12%`
- `consecutive_losses >= 8`
- `news_severity = "CRITICAL"`
- `exchange_status = "MAINTENANCE" OR "OUTAGE"`
- `api_error_rate_5min > 20%`
- `manual_kill_switch = TRUE`

**Limites R3:**

| Parâmetro | Limite | Nota |
|-----------|--------|------|
| `max_position_pct` | 0% | Sem novas posições |
| `max_leverage` | 1x | Sem alavancagem |
| `entries_allowed` | FALSE | Kill completo |
| `only_exits` | TRUE | Só permite fechar |

**Ações:**
- ALLOW: Saídas de emergência (sempre)
- PAUSE: Todas as operações (exceto saídas)
- DENY: Entradas, aumentos, alavancagem

> **IMPORTANTE**: Saída de R3 requer confirmação humana explícita. O sistema NÃO retorna automaticamente a R2 ou inferior.

---

## 2. Regras Determinísticas (12+ obrigatórias)

Formato:
- `RULE_ID`: Identificador único
- `IF`: Condições objetivas (booleanas/numéricas)
- `THEN`: Verdict + constraints
- `EVIDENCE REQUIRED`: O que deve existir para auditar
- `FAIL-CLOSED`: O que acontece se faltar dado

---

### RULE-FIN-001: Volatilidade Extrema (ATR)

```yaml
RULE_ID: RULE-FIN-001
TRIGGER: ATR-based volatility
IF:
  - ATR_14 >= ATR_baseline * 4.0
  - OR: return_1min_abs >= 2.0%
THEN:
  verdict: RESTRICT
  constraints:
    policy_level: R2 (se ATR < 8x) | R3 (se ATR >= 8x)
    max_position_pct: 2%
    pause_entries_sec: 300
    require_confirmation: true
EVIDENCE_REQUIRED:
  - source: "exchange:binance"
  - metric: "ATR_14"
  - metric: "return_1min"
  - ts: "<ISO8601>"
FAIL_CLOSED:
  - IF ATR_14 unavailable: assume R2
  - IF return_1min unavailable: assume R2
```

**Calibração (DEFAULT PROVISÓRIO):**
- `ATR_baseline`: Média ATR_14 dos últimos 30 dias
- Recalcular semanalmente via telemetria

---

### RULE-FIN-002: Spread/Slippage Anômalo

```yaml
RULE_ID: RULE-FIN-002
TRIGGER: Spread ou slippage anormal
IF:
  - spread_bps >= 30
  - OR: last_slippage_bps > max_slippage_bps
THEN:
  verdict: RESTRICT
  constraints:
    policy_level: R2
    pause_entries_sec: 60
    max_slippage_bps: 5
    require_confirmation: true (se spread >= 50)
EVIDENCE_REQUIRED:
  - source: "exchange:binance"
  - metric: "bid_ask_spread_bps"
  - metric: "last_slippage_bps"
  - ts: "<ISO8601>"
FAIL_CLOSED:
  - IF spread unavailable: DENY entry
  - IF slippage unavailable: DENY entry
```

**Calibração (DEFAULT PROVISÓRIO):**
- Thresholds baseados em percentil 95 histórico
- Atualizar após 1000 trades

---

### RULE-FIN-003: Latência/Desconexão WebSocket

```yaml
RULE_ID: RULE-FIN-003
TRIGGER: Latência alta ou desconexão
IF:
  - ws_latency_ms >= 500
  - OR: ws_disconnected = TRUE
  - OR: last_heartbeat_age_sec > 30
THEN:
  verdict: PAUSE | DENY
  constraints:
    policy_level: R2 (latency >= 500) | R3 (disconnect)
    entries_allowed: false
    only_exits: true
EVIDENCE_REQUIRED:
  - source: "infra:websocket"
  - metric: "ws_latency_ms"
  - metric: "ws_connected"
  - metric: "last_heartbeat_ts"
  - ts: "<ISO8601>"
FAIL_CLOSED:
  - IF ws_latency unavailable: assume R3
  - IF heartbeat stale > 60s: assume R3
```

---

### RULE-FIN-004: Rate Limit / API Instável

```yaml
RULE_ID: RULE-FIN-004
TRIGGER: Rate limit próximo ou API com erros
IF:
  - api_rate_limit_remaining <= 20%
  - OR: api_error_rate_5min > 5%
THEN:
  verdict: RESTRICT | PAUSE
  constraints:
    policy_level: R1 (20%) | R2 (10%) | R3 (5% ou error > 20%)
    max_trades_per_day: -50%
    pause_api_calls_sec: 60
EVIDENCE_REQUIRED:
  - source: "exchange:binance"
  - metric: "rate_limit_remaining_pct"
  - metric: "api_error_rate_5min"
  - ts: "<ISO8601>"
FAIL_CLOSED:
  - IF rate_limit unavailable: assume R2
  - IF error_rate unavailable: pause 60s + retry
```

---

### RULE-FIN-005: Drawdown Diário/Semanal

```yaml
RULE_ID: RULE-FIN-005
TRIGGER: Drawdown excedido
IF:
  - daily_drawdown_pct >= 5%
  - OR: weekly_drawdown_pct >= 10%
THEN:
  verdict: PAUSE | DENY
  constraints:
    policy_level: R2 (5%/10%) | R3 (8%/12%)
    entries_allowed: false (R3)
    only_exits: true (R3)
    session_paused_until: "next_day" (daily) | "next_week" (weekly)
EVIDENCE_REQUIRED:
  - source: "portfolio:local"
  - metric: "daily_pnl_pct"
  - metric: "weekly_pnl_pct"
  - metric: "initial_balance"
  - ts: "<ISO8601>"
FAIL_CLOSED:
  - IF pnl unavailable: assume R3 + REQUIRE_CONFIRMATION
```

**Calibração (DEFAULT PROVISÓRIO):**
- `daily_limit`: 5% (R2), 8% (R3)
- `weekly_limit`: 10% (R2), 12% (R3)
- Ajustar via simulação de Monte Carlo

---

### RULE-FIN-006: Sequência de Perdas

```yaml
RULE_ID: RULE-FIN-006
TRIGGER: Múltiplas perdas consecutivas
IF:
  - consecutive_losses >= 4
THEN:
  verdict: RESTRICT | PAUSE
  constraints:
    policy_level: R1 (4-5) | R2 (6-7) | R3 (8+)
    max_position_pct: -50% por nível
    pause_entries_sec: 300 * (consecutive_losses - 3)
EVIDENCE_REQUIRED:
  - source: "portfolio:local"
  - metric: "consecutive_losses"
  - metric: "last_trade_results[]"
  - ts: "<ISO8601>"
FAIL_CLOSED:
  - IF trade_history unavailable: assume R2
```

---

### RULE-FIN-007: Evento Externo (News)

```yaml
RULE_ID: RULE-FIN-007
TRIGGER: Notícia de alta severidade
IF:
  - news_severity IN ["HIGH", "CRITICAL"]
  - AND: news_confidence >= 0.6
  - AND: news_relevance IN ["BTC", "ETH", "CRYPTO", "MACRO"]
THEN:
  verdict: RESTRICT | REQUIRE_CONFIRMATION
  constraints:
    policy_level: R1 (HIGH, conf >= 0.8) | R2 (HIGH, conf < 0.8) | R3 (CRITICAL)
    require_confirmation: true
    pause_entries_sec: 600
    max_position_pct: 2%
EVIDENCE_REQUIRED:
  - source: "news:<provider>"
  - metric: "severity"
  - metric: "confidence"
  - metric: "relevance_tags[]"
  - metric: "headline"
  - ts: "<ISO8601>"
FAIL_CLOSED:
  - IF news unavailable: continue (não é bloqueante)
  - IF confidence unavailable: assume confidence = 0.5
```

> **NOTA**: Notícia NÃO gera sinal de trade. Apenas restringe operações até confirmação.

---

### RULE-FIN-008: Require Confirmation Mode

```yaml
RULE_ID: RULE-FIN-008
TRIGGER: Operação requer confirmação humana
IF:
  - policy_level >= R1
  - AND: operation_type IN ["entry", "increase_position"]
  - AND: (position_value_pct > 2% OR require_confirmation = TRUE)
THEN:
  verdict: REQUIRE_CONFIRMATION
  constraints:
    confirmation_timeout_sec: 300
    confirmation_type: "price_check" | "human_ack"
    auto_deny_on_timeout: true
EVIDENCE_REQUIRED:
  - source: "abs:governance"
  - metric: "confirmation_requested"
  - metric: "confirmation_type"
  - metric: "requester_context"
  - ts: "<ISO8601>"
FAIL_CLOSED:
  - IF confirmation_timeout: DENY
  - IF confirmation_unavailable: DENY
```

---

### RULE-FIN-009: Janela de Liquidez

```yaml
RULE_ID: RULE-FIN-009
TRIGGER: Horário de baixa liquidez
IF:
  - hour_utc IN [0..3] OR hour_utc IN [10..12] # Exemplo: weekends, feriados
  - OR: is_weekend = TRUE
  - OR: volume_24h < volume_baseline * 0.3
THEN:
  verdict: RESTRICT
  constraints:
    policy_level: R1
    max_position_pct: 5%
    max_slippage_bps: 10
    require_confirmation: true (para > 3% capital)
EVIDENCE_REQUIRED:
  - source: "exchange:binance"
  - metric: "hour_utc"
  - metric: "is_weekend"
  - metric: "volume_24h"
  - ts: "<ISO8601>"
FAIL_CLOSED:
  - IF volume unavailable: assume low liquidity (R1)
```

**Calibração (DEFAULT PROVISÓRIO):**
- `volume_baseline`: Média móvel 7 dias
- Horários podem variar por ativo

---

### RULE-FIN-010: Exposição Máxima por Ativo e Total

```yaml
RULE_ID: RULE-FIN-010
TRIGGER: Exposição excede limites
IF:
  - position_pct[asset] > max_position_pct
  - OR: total_exposure_pct > max_total_exposure_pct
THEN:
  verdict: DENY
  constraints:
    action_denied: "increase_position"
    only_reduces_allowed: true
EVIDENCE_REQUIRED:
  - source: "portfolio:local"
  - metric: "position_pct[asset]"
  - metric: "total_exposure_pct"
  - metric: "max_position_pct" (from policy_level)
  - ts: "<ISO8601>"
FAIL_CLOSED:
  - IF position unavailable: DENY all entries
```

**Limites (DEFAULT PROVISÓRIO):**
- `max_position_pct` por ativo: 10% (R0), 5% (R1), 2% (R2), 0% (R3)
- `max_total_exposure_pct`: 30% (R0), 15% (R1), 5% (R2), 0% (R3)

---

### RULE-FIN-011: Limites Diários (Trades/Ordens)

```yaml
RULE_ID: RULE-FIN-011
TRIGGER: Excesso de atividade
IF:
  - trades_today >= max_trades_per_day
  - OR: orders_cancelled_today >= max_orders_cancelled
THEN:
  verdict: PAUSE | DENY
  constraints:
    pause_until: "next_day"
    entries_allowed: false
    only_exits: true
EVIDENCE_REQUIRED:
  - source: "portfolio:local"
  - metric: "trades_today"
  - metric: "orders_cancelled_today"
  - ts: "<ISO8601>"
FAIL_CLOSED:
  - IF counts unavailable: assume limit reached (DENY)
```

---

### RULE-FIN-012: Modo Read-Only (Scanner)

```yaml
RULE_ID: RULE-FIN-012
TRIGGER: Modo read-only ativo
IF:
  - mode = "read_only" OR mode = "scanner"
  - OR: manual_read_only = TRUE
THEN:
  verdict: DENY
  constraints:
    entries_allowed: false
    exits_allowed: false
    orders_allowed: false
    data_collection: true
EVIDENCE_REQUIRED:
  - source: "config:local"
  - metric: "mode"
  - metric: "manual_read_only"
  - ts: "<ISO8601>"
FAIL_CLOSED:
  - IF mode unavailable: assume read_only (safest)
```

---

### RULE-FIN-013: Exchange Status Degradado

```yaml
RULE_ID: RULE-FIN-013
TRIGGER: Exchange com problemas
IF:
  - exchange_status != "OPERATIONAL"
THEN:
  verdict: PAUSE | DENY
  constraints:
    policy_level: R2 (DEGRADED) | R3 (MAINTENANCE/OUTAGE)
    entries_allowed: false (R3)
    only_exits: true
EVIDENCE_REQUIRED:
  - source: "exchange:binance:status"
  - metric: "status"
  - metric: "status_message"
  - ts: "<ISO8601>"
FAIL_CLOSED:
  - IF status unavailable: assume DEGRADED (R2)
```

---

### RULE-FIN-014: Kill-Switch Manual

```yaml
RULE_ID: RULE-FIN-014
TRIGGER: Kill-switch ativado manualmente
IF:
  - manual_kill_switch = TRUE
THEN:
  verdict: DENY
  constraints:
    policy_level: R3
    entries_allowed: false
    exits_allowed: true (emergency only)
    require_human_to_resume: true
EVIDENCE_REQUIRED:
  - source: "config:manual"
  - metric: "kill_switch"
  - metric: "activated_by"
  - metric: "activated_at"
  - ts: "<ISO8601>"
FAIL_CLOSED:
  - N/A (trigger é manual)
```

---

## 3. Decision Envelope (Schema v1 — JSON)

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "ABS Finance Decision Envelope v1",
  "type": "object",
  "required": [
    "decision_id",
    "ts",
    "scope_assets",
    "verdict",
    "policy_level",
    "constraints",
    "reason",
    "evidence",
    "expires_at",
    "chain_hash"
  ],
  "properties": {
    "decision_id": {
      "type": "string",
      "format": "uuid",
      "description": "UUID único da decisão"
    },
    "ts": {
      "type": "string",
      "format": "date-time",
      "description": "Timestamp ISO8601 da decisão"
    },
    "scope_assets": {
      "type": "array",
      "items": { "type": "string" },
      "description": "Ativos afetados (ex: ['BTC-PERP', 'ETH-PERP'])"
    },
    "verdict": {
      "type": "string",
      "enum": ["ALLOW", "RESTRICT", "PAUSE", "REQUIRE_CONFIRMATION", "DENY"],
      "description": "Resultado da avaliação"
    },
    "policy_level": {
      "type": "string",
      "enum": ["R0", "R1", "R2", "R3"],
      "description": "Nível de política ativo"
    },
    "constraints": {
      "type": "object",
      "properties": {
        "max_position_pct": { "type": "number" },
        "max_leverage": { "type": "number" },
        "max_slippage_bps": { "type": "number" },
        "max_trades_per_day": { "type": "integer" },
        "pause_entries_sec": { "type": "integer" },
        "require_confirmation": { "type": "boolean" },
        "only_exits": { "type": "boolean" },
        "entries_allowed": { "type": "boolean" }
      }
    },
    "reason": {
      "type": "string",
      "maxLength": 500,
      "description": "Motivo verificável e curto"
    },
    "evidence": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["source", "metric", "value", "ts"],
        "properties": {
          "source": { "type": "string" },
          "metric": { "type": "string" },
          "value": { "type": ["string", "number", "boolean"] },
          "ts": { "type": "string", "format": "date-time" }
        }
      }
    },
    "expires_at": {
      "type": "string",
      "format": "date-time",
      "description": "Expiração obrigatória do envelope"
    },
    "chain_hash": {
      "type": "string",
      "description": "Hash do envelope anterior (para auditoria encadeada)"
    },
    "triggered_rules": {
      "type": "array",
      "items": { "type": "string" },
      "description": "IDs das regras que dispararam"
    }
  }
}
```

---

### Exemplo 1: Notícia de Alta Severidade + Baixa Confiança

**Cenário**: Notícia de petróleo com severidade HIGH mas confiança 0.65

```json
{
  "decision_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "ts": "2026-01-26T14:30:00Z",
  "scope_assets": ["BTC-PERP", "ETH-PERP"],
  "verdict": "RESTRICT",
  "policy_level": "R2",
  "constraints": {
    "max_position_pct": 2,
    "max_leverage": 2,
    "max_slippage_bps": 5,
    "pause_entries_sec": 600,
    "require_confirmation": true,
    "entries_allowed": false
  },
  "reason": "Notícia HIGH severity (petróleo) com confidence 0.65 < 0.8 → R2 + REQUIRE_CONFIRMATION",
  "evidence": [
    {
      "source": "news:reuters",
      "metric": "severity",
      "value": "HIGH",
      "ts": "2026-01-26T14:28:00Z"
    },
    {
      "source": "news:reuters",
      "metric": "confidence",
      "value": 0.65,
      "ts": "2026-01-26T14:28:00Z"
    },
    {
      "source": "news:reuters",
      "metric": "headline",
      "value": "OPEC+ anuncia corte surpresa de produção",
      "ts": "2026-01-26T14:28:00Z"
    },
    {
      "source": "news:reuters",
      "metric": "relevance_tags",
      "value": ["MACRO", "COMMODITIES"],
      "ts": "2026-01-26T14:28:00Z"
    }
  ],
  "expires_at": "2026-01-26T15:30:00Z",
  "chain_hash": "sha256:abc123def456...",
  "triggered_rules": ["RULE-FIN-007"]
}
```

---

### Exemplo 2: Drawdown Diário Excedido

**Cenário**: Drawdown diário atingiu 8.5% → R3 ativado

```json
{
  "decision_id": "f9e8d7c6-b5a4-3210-fedc-ba0987654321",
  "ts": "2026-01-26T18:45:00Z",
  "scope_assets": ["BTC-PERP", "ETH-PERP"],
  "verdict": "DENY",
  "policy_level": "R3",
  "constraints": {
    "max_position_pct": 0,
    "max_leverage": 1,
    "max_slippage_bps": 0,
    "pause_entries_sec": 86400,
    "require_confirmation": true,
    "only_exits": true,
    "entries_allowed": false
  },
  "reason": "Daily drawdown 8.5% >= 8% threshold → R3 KILL-SWITCH. Apenas saídas permitidas.",
  "evidence": [
    {
      "source": "portfolio:local",
      "metric": "daily_drawdown_pct",
      "value": 8.5,
      "ts": "2026-01-26T18:44:00Z"
    },
    {
      "source": "portfolio:local",
      "metric": "initial_balance",
      "value": 10000,
      "ts": "2026-01-26T00:00:00Z"
    },
    {
      "source": "portfolio:local",
      "metric": "current_balance",
      "value": 9150,
      "ts": "2026-01-26T18:44:00Z"
    },
    {
      "source": "config:thresholds",
      "metric": "daily_drawdown_limit_r3",
      "value": 8,
      "ts": "2026-01-26T00:00:00Z"
    }
  ],
  "expires_at": "2026-01-27T00:00:00Z",
  "chain_hash": "sha256:789xyz123...",
  "triggered_rules": ["RULE-FIN-005"]
}
```

---

## 4. Checklist de Auditoria (PASS/FAIL)

Use esta checklist para validar o Policy Set antes de ativação:

| # | Critério | Status | Evidência |
|---|----------|--------|-----------|
| 1 | Todas as regras são determinísticas? (sem ML, sem probabilístico) | ☐ PASS / ☐ FAIL | Verificar: nenhum `predict()`, `model.infer()` |
| 2 | Existe fail-closed para ausência de dados? | ☐ PASS / ☐ FAIL | Verificar: toda regra tem `FAIL_CLOSED` |
| 3 | Existe kill-switch automático (R3)? | ☐ PASS / ☐ FAIL | Verificar: RULE-FIN-014, condições R3 |
| 4 | Regras têm métricas e thresholds explícitos? | ☐ PASS / ☐ FAIL | Verificar: nenhum "muito alto", "baixo demais" |
| 5 | Envelope tem `expires_at` obrigatório? | ☐ PASS / ☐ FAIL | Verificar: schema JSON |
| 6 | Há limites de perda (drawdown)? | ☐ PASS / ☐ FAIL | Verificar: RULE-FIN-005 |
| 7 | Há limites de alavancagem? | ☐ PASS / ☐ FAIL | Verificar: `max_leverage` em cada R-level |
| 8 | Existe separação clara: ABS governa, script executa? | ☐ PASS / ☐ FAIL | Verificar: ABS não emite ordens |
| 9 | Evidence é estruturada (source, metric, value, ts)? | ☐ PASS / ☐ FAIL | Verificar: schema de evidence |
| 10 | Saída de R3 requer confirmação humana? | ☐ PASS / ☐ FAIL | Verificar: R3 exit criteria |
| 11 | Thresholds têm justificativa ou indicam "PROVISÓRIO"? | ☐ PASS / ☐ FAIL | Verificar: notas de calibração |
| 12 | Chain_hash está presente para auditoria encadeada? | ☐ PASS / ☐ FAIL | Verificar: schema JSON |

---

## 5. Anti-Patterns (Práticas Proibidas)

| # | Anti-Pattern | Por que é proibido | Confirmação |
|---|--------------|-------------------|-------------|
| 1 | **"Notícia gera trade"** | Notícia é input não-confiável. ABS RESTRINGE, não COMPRA/VENDE. | ☐ Confirmado |
| 2 | **"ABS decide entrada/saída"** | ABS governa permissões. Script decide sinais. Separação obrigatória. | ☐ Confirmado |
| 3 | **"Sem limites de perda"** | Sistema sem drawdown limits vai à ruína. RULE-FIN-005 obrigatória. | ☐ Confirmado |
| 4 | **"Sem expiração do envelope"** | Envelope eterno = governança obsoleta. `expires_at` obrigatório. | ☐ Confirmado |
| 5 | **"Fail-open em dados ausentes"** | Se dado falta, assumir pior caso (fail-closed). Nunca "permitir por default". | ☐ Confirmado |
| 6 | **"Threshold sem justificativa"** | Todo número deve ter base (backtest, percentil, ou "PROVISÓRIO + plano de calibração"). | ☐ Confirmado |
| 7 | **"Decisão probabilística no ABS"** | ABS é determinístico. Sem `confidence > 0.7 → allow`. Use thresholds fixos. | ☐ Confirmado |
| 8 | **"Retorno automático de R3"** | Kill-switch (R3) só desliga com confirmação humana. Nunca automático. | ☐ Confirmado |
| 9 | **"Alavancagem sem limite"** | Alavancagem ilimitada = ruína garantida. Limitar em cada R-level. | ☐ Confirmado |
| 10 | **"Overtrading permitido"** | Sem limite de trades/dia = risco de loop. RULE-FIN-011 obrigatória. | ☐ Confirmado |

---

## Referências

- [Policy Pack v0](../decisions/policy_pack_v0.md) — Bots operacionais
- [Decision Contract v0](../decisions/decision_contract_v0.md) — Schema base
- [ADR-008](../decisions/) — Decision Envelope v1

---

## Notas de Calibração

> **TODOS os thresholds numéricos são DEFAULT PROVISÓRIO.**

**Plano de calibração:**

1. **Volatilidade (ATR)**:
   - Calcular `ATR_baseline` = média ATR_14 dos últimos 30 dias
   - Recalcular semanalmente
   - Ajustar multiplicadores (2x, 4x, 8x) após 1000 trades

2. **Spread/Slippage**:
   - Usar percentil 95 histórico como baseline
   - Atualizar após 1000 trades

3. **Drawdown**:
   - Valores iniciais (5%, 8%, 10%, 12%) baseados em simulação conservadora
   - Ajustar via Monte Carlo com histórico de 6 meses

4. **Consecutive Losses**:
   - Valores (4, 6, 8) baseados em "rule of thumb" conservador
   - Ajustar após análise de distribuição de perdas

5. **Volume/Liquidez**:
   - `volume_baseline` = média móvel 7 dias
   - Atualizar diariamente

---

**Versão**: 1.0.0  
**Data**: 2026-01-26  
**Autor**: ABS Finance Policy Team  
**Status**: DRAFT — Aguardando revisão e calibração
