# RFC-001: Camada de Persistência Agnóstica (Edge First)

## 1. Contexto & Problema

O auditor identificou dois riscos existenciais no ABS Core v1:
1.  **Blocker de Edge**: A dependência direta de `better-sqlite3` (C++ bindings) impede o deploy em Cloudflare Workers "Raw", quebrando a tese "Scale-to-Zero".
2.  **Segurança com Amnésia**: A `RiskTelemetry` em memória permite ataques de "Crash Reset" (derrubar o processo para zerar a reputação).

## 2. Objetivo

Desacoplar o Kernel do ABS de implementações síncronas de banco de dados, introduzindo uma interface `IPersistenceLayer` puramente assíncrona que suporte:
- **Cloudflare D1** (via bindings HTTP/Wasm).
- **Turso/LibSQL** (via HTTP).
- **SQLite Local** (para dev/testes, encapsulado).

## 3. A Nova Interface (`IPersistenceLayer`)

A interface deve ser agnóstica ao driver subjacente e forçar assincronicidade em todas as operações (preparando para chamadas de rede/RPC).

```typescript
export interface IPersistenceLayer {
  /**
   * Executa uma query de escrita/leitura genérica.
   * Retorna um array de resultados tipado.
   */
  query<T = any>(sql: string, params?: any[]): Promise<T[]>;

  /**
   * Executa uma query de modificação (INSERT, UPDATE, DELETE).
   * Retorna metadados da execução.
   */
  execute(sql: string, params?: any[]): Promise<ExecutionResult>;

  /**
   * Operações de KV Store (para RiskTelemetry e Sessão).
   * Deve ser implementado usando tabelas SQL ou KV nativo (Workers KV).
   */
  kv: {
    get<T>(key: string): Promise<T | null>;
    set(key: string, value: any, ttl?: number): Promise<void>;
    delete(key: string): Promise<void>;
  };
    
  /**
   * Gerenciamento de transações (Batching).
   * Essencial para D1 que suporta batching atômico.
   */
  batch(statements: Array<{ sql: string; params?: any[] }>): Promise<ExecutionResult[]>;
}

export interface ExecutionResult {
  success: boolean;
  meta?: {
    changes?: number;
    last_row_id?: string | number;
    duration_ms?: number;
  };
}
```

## 4. Estratégia de Implementação

### 4.1. Adapters

Criaremos um padrão Factory para instanciar o adapter correto baseado em `ABS_RUNTIME_ENV`:

1.  **`D1Adapter` (`infra/adapters/d1.ts`)**:
    - Usa `env.DB.prepare().bind().all()` do Cloudflare Workers.
    - Suporta `batch()` nativo do D1.
    - Zero dependências de Node.js `fs` ou `path`.

2.  **`SQLiteAdapter` (`infra/adapters/sqlite.ts`)**:
    - Mantém `better-sqlite3` mas encapsulado em Promises.
    - Exclusivo para ambiente `NODE_ENV=dev` ou `test`.

### 4.2. Migração da RiskTelemetry (Correção INV-004)

Atualmente, `RiskTelemetry` armazena scores em um `Map<string, number>`.
A nova implementação injetará `IPersistenceLayer` no construtor:

```typescript
// Antes
private riskScores = new Map<string, number>();

// Depois
constructor(private db: IPersistenceLayer) {}

async recordAnomaly(agentId: string, severity: number) {
  const key = `risk:${agentId}`;
  const current = await this.db.kv.get<number>(key) || 0;
  const newScore = current + severity;
  
  // Persistência com TTL longo (ex: 24h) para evitar "esquecimento" rápido
  await this.db.kv.set(key, newScore, 86400); 
}
```

 Isso elimina o vetor de ataque "Crash Reset". Mesmo se o container cair, o score está no D1/KV.

## 5. Plano de Ação

1.  **Refactor Core**: Mover `IPersistenceLayer` para `packages/core/src/infra/persistence.ts`.
2.  **Implementar D1 Adapter**: Criar `packages/core/src/infra/adapters/d1-adapter.ts`.
3.  **Atualizar ABSPolicyEngine**: Remover dependência direta de classes concretas de DB.
4.  **Migrar RiskTelemetry**: Alterar armazenamento para `db.kv`.

## 6. Benefícios

- **Edge Native**: Destrava deploy em centenas de PoPs da Cloudflare.
- **Segurança Antifráfgy**: Agentes não podem limpar a ficha derrubando o servidor.
- **Escala**: D1 escala leituras globalmente.
