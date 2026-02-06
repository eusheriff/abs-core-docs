# ABS Finance Evidence Model v1

> **Versão**: 1.0.0  
> **Domínio**: Finance / Algorithmic Trading  
> **Princípio**: ABS não coleta dados. O Executor (Script) envia pacotes de evidência normalizados para avaliação.

---

## 1. Evidence Types

Definição dos tipos de payloads aceitos pelo ABS para avaliação de risco.

### Estrutura Base
Todas as evidências devem seguir este envelope base:

```typescript
interface EvidencePacket {
  type: EvidenceType;
  source: string;        // ex: "binance", "portfolio:local"
  ts: string;            // ISO8601 (emission time)
  ttl_ms: number;        // Tempo de vida da evidência (staleness)
  payload: Record<string, any>;
  signature?: string;    // HMAC-SHA256(type + ts + payload)
}
```

### Catálogo de Tipos

#### MARKET_TICK
Dados de preço em tempo real.
- **Payload**: `{ symbol: string, price: number, quantity: number, trade_id: string }`
- **Resolução**: Tick-by-tick (streaming) ou Agregado (100ms)
- **Fonte**: WebSocket Exchange

#### OHLCV_BAR
Barras de preço fechadas.
- **Payload**: `{ symbol: string, interval: "1m"|"5m"|"1h", o: number, h: number, l: number, c: number, v: number, closed: boolean }`
- **Resolução**: 1 minuto (mínimo)
- **Fonte**: WebSocket/REST Exchange

#### ORDERBOOK_TOP
Melhor bid/ask para cálculo de spread.
- **Payload**: `{ symbol: string, bid_price: number, bid_qty: number, ask_price: number, ask_qty: number }`
- **Resolução**: Snapshot (ex: 1s)
- **Fonte**: WebSocket Exchange

#### SPREAD_SNAPSHOT
Métrica derivada pré-calculada (opcional, se script já calcular).
- **Payload**: `{ symbol: string, spread_bps: number, depth_impact_bps: number }`
- **Resolução**: Snapshot
- **Fonte**: Executor (Calculated)

#### LATENCY_SNAPSHOT
Métricas de saúde da conexão.
- **Payload**: `{ connection_id: string, rtt_ms: number, jitter_ms: number, last_heartbeat_age_ms: number }`
- **Resolução**: Ping interval (ex: 5s)
- **Fonte**: Infra Monitor

#### API_HEALTH
Estado da API REST.
- **Payload**: `{ endpoint: string, status: "OK"|"DEGRADED"|"DOWN", error_rate_5min: number, avg_response_ms: number }`
- **Resolução**: 1 min
- **Fonte**: Infra Monitor

#### RATE_LIMIT_EVENT
Consumo de quota da API.
- **Payload**: `{ resource: "IP"|"UID", limit_type: "REQUESTS"|"ORDERS", remaining: number, reset_in_ms: number }`
- **Resolução**: On-change
- **Fonte**: API Headers Interceptor

#### NEWS_EVENT
Eventos externos normalizados.
- **Payload**: `{ provider: string, headline: string, severity: "LOW"|"MEDIUM"|"HIGH"|"CRITICAL", confidence: number (0-1), relevance_tags: string[] }`
- **Resolução**: Event-driven
- **Fonte**: News Aggregator / Manual Input

#### POSITION_SNAPSHOT
Estado atual da carteira.
- **Payload**: `{ asset: string, position_amt: number, entry_price: number, unrealized_pnl: number, leverage: number, liquidation_price: number }`
- **Resolução**: On-change / Periodic (10s)
- **Fonte**: Exchange Account API

#### PNL_SNAPSHOT
Performance histórica recente.
- **Payload**: `{ balance_initial: number, balance_current: number, daily_pnl_pct: number, weekly_pnl_pct: number, consecutive_losses: number, trades_today: number }`
- **Resolução**: Periodic (1 min)
- **Fonte**: Portfolio Manager (Local DB)

---

## 2. Metrics Catalog (Canonical Names)

Definições padronizadas para evitar ambiguidade (ex: "return" vs "log return").

| Métrica | Definição / Cálculo Conceitual | Unidade | Janela Padrão |
|---------|--------------------------------|---------|---------------|
| `atr_1m` | Average True Range (14 periodos de 1m) | Price | 14 min |
| `ret_1m` | `(close_t / close_t-1) - 1` | Decimal | 1 min |
| `ret_5m` | `(close_t / close_t-5) - 1` | Decimal | 5 min |
| `spread_bps` | `((ask - bid) / mid) * 10000` | BPS | Snapshot |
| `slippage_bps` | `abs((fill_price - target_price) / target_price) * 10000` | BPS | Last Trade |
| `ws_rtt_ms` | WebSocket Round-Trip Time (Ping-Pong) | ms | Inst |
| `api_rtt_ms` | REST API Latency (Request-Response) | ms | Inst |
| `disconnect_count` | Nº de reconexões forçadas na janela | Int | 1h |
| `dd_day_pct` | `(peak_balance_day - current_balance) / peak_balance_day` | Decimal | Trading Day |
| `dd_week_pct` | `(peak_balance_week - current_balance) / peak_balance_week` | Decimal | Rolling 7d |
| `loss_streak_n` | Nº de trades consecutivos com PnL < 0 | Int | Histórico |
| `trade_count_day` | Nº de trades executados desde 00:00 UTC | Int | Trading Day |
| `position_notional` | `abs(position_amt * mark_price)` | USD | Snapshot |
| `leverage_effective`| `total_position_notional / account_equity` | Float | Snapshot |

---

## 3. Integrity + Non-Repudiation

Para garantir que o ABS avalia dados reais e não manipulados (ou atrasados):

### 3.1. Assinatura (HMAC)
Todo pacote crítico de evidência (ex: PnL, Posição) deve ser assinado pelo Executor.
- **Chave**: Compartilhada (`ABS_SECRET_KEY`).
- **Algoritmo**: `HMAC-SHA256(ts + type + JSON.stringify(payload))`.
- **Validação**: ABS rejeita pacotes com assinatura inválida.

### 3.2. Staleness & TTL
ABS rejeita evidências "velhas" para evitar replay attacks ou decisões em stale data.
- **Regra**: `now() - evidence.ts > evidence.ttl_ms → REJECT`.
- **TTL Padrão**:
  - Market Data: 1000ms (1s)
  - Infra Data: 5000ms (5s)
  - PnL/Position: 10000ms (10s)
  - News: 1 hora

### 3.3. Referência no Decision Envelope
O ABS **não armazena** o payload bruto, mas registra o "Receipt" da evidência no Decision Log.
- O campo `evidence` no Decision Envelope contém apenas os metadados e o hash do payload original, permitindo auditoria futura se o log bruto for guardado externamente.

---

## 4. FAIL-CLOSED Matrix

Define o comportamento do ABS quando uma evidência obrigatória está **ausente** ou **stale** (expirada).

| Rule Class | Evidence Types | Dados Obrigatórios | Se Faltar/Stale | Impacto |
|------------|----------------|--------------------|-----------------|---------|
| **INFRA** | `LATENCY_SNAPSHOT`, `API_HEALTH` | `ws_rtt_ms` | **DENY** (Kill) | Sem conectividade = Sem trade. |
| **MARKET** | `OHLCV_BAR`, `ORDERBOOK_TOP` | `atr_14`, `spread_bps` | **RESTRICT** (R2) | Assume alta volatilidade. |
| **RISK** | `PNL_SNAPSHOT`, `POSITION_SNAPSHOT` | `dd_day_pct`, `leverage` | **DENY** (Kill) | Sem saber PnL = Risco infinito. |
| **API** | `RATE_LIMIT_EVENT` | `remaining` | **RESTRICT** (R1) | Assume rate limit baixo. |
| **NEWS** | `NEWS_EVENT` | `severity` | **IGNORE** (Fail-Open*) | Notícias são opcionais (melhoria). |

> (*) **News Exception**: Único caso de "Fail-Open". A ausência de notícias não impede o trading, pois o sistema opera puramente técnico. A presença de notícias apenas *adiciona* restrições.

---

## 5. Exemplos

### Exemplo 1: Pacote de Evidência (Infra + Risco) 
JSON enviado pelo Executor para o ABS avaliar.

```json
{
  "packet_id": "evt_123456789",
  "ts": "2026-01-26T14:30:05.123Z",
  "source": "executor:machine_01",
  "items": [
    {
      "type": "LATENCY_SNAPSHOT",
      "ttl_ms": 5000,
      "payload": {
        "rtt_ms": 150,
        "last_heartbeat_age_ms": 200
      }
    },
    {
      "type": "PNL_SNAPSHOT",
      "ttl_ms": 10000,
      "payload": {
        "daily_pnl_pct": -0.045,  // -4.5%
        "consecutive_losses": 3,
        "trades_today": 12
      }
    }
  ],
  "signature": "hmac_sha256_signature_here..."
}
```

### Exemplo 2: Decisão RESTRICT (Resposta do ABS)
ABS avalia o pacote acima e retorna uma restrição (Daily Drawdown próximo do limite de 5%).

```json
{
  "decision_id": "dec_987654321",
  "ts": "2026-01-26T14:30:05.150Z",
  "verdict": "RESTRICT",
  "policy_level": "R1",
  "constraints": {
    "max_position_pct": 5,
    "max_leverage": 3,
    "max_trades_per_day": 25,
    "reason_code": "WARN_DRAWDOWN_NEAR_LIMIT"
  },
  "reason": "Daily drawdown -4.5% approaches R2 limit (-5%). Scaling down leverage.",
  "evidence": [
    { "source": "portfolio", "metric": "daily_pnl_pct", "value": -0.045, "ts": "..." }
  ],
  "expires_at": "2026-01-26T14:31:00Z" // Validade curta (1 min)
}
```
