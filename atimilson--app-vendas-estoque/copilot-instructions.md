## app-vendas-estoque

> Estrutura e padrões do banco de dados SQLite


# Banco de Dados - SQLite

## 📊 Estrutura de Tabelas

### Tabela: users
```sql
CREATE TABLE IF NOT EXISTS users (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  email TEXT UNIQUE NOT NULL,
  senha TEXT NOT NULL,
  nome TEXT NOT NULL,
  ativo INTEGER DEFAULT 1,
  created_at TEXT DEFAULT CURRENT_TIMESTAMP,
  updated_at TEXT DEFAULT CURRENT_TIMESTAMP
);
```

### Tabela: produtos
```sql
CREATE TABLE IF NOT EXISTS produtos (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  nome TEXT NOT NULL,
  preco_compra REAL NOT NULL,
  preco_venda REAL NOT NULL,
  quantidade_total INTEGER DEFAULT 0,
  foto TEXT,
  ativo INTEGER DEFAULT 1,
  created_at TEXT DEFAULT CURRENT_TIMESTAMP,
  updated_at TEXT DEFAULT CURRENT_TIMESTAMP
);
```

### Tabela: codigos_barras
```sql
CREATE TABLE IF NOT EXISTS codigos_barras (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  produto_id INTEGER NOT NULL,
  codigo TEXT NOT NULL,
  created_at TEXT DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (produto_id) REFERENCES produtos(id) ON DELETE CASCADE,
  UNIQUE(produto_id, codigo)
);
```

### Tabela: lotes
```sql
CREATE TABLE IF NOT EXISTS lotes (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  produto_id INTEGER NOT NULL,
  numero_lote TEXT NOT NULL,
  data_validade TEXT,
  quantidade INTEGER NOT NULL,
  ativo INTEGER DEFAULT 1,
  created_at TEXT DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (produto_id) REFERENCES produtos(id) ON DELETE CASCADE
);
```

### Tabela: entradas
```sql
CREATE TABLE IF NOT EXISTS entradas (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  data_entrada TEXT NOT NULL,
  data_emissao_nota TEXT,
  valor_total REAL NOT NULL,
  fornecedor TEXT,
  observacao TEXT,
  created_at TEXT DEFAULT CURRENT_TIMESTAMP
);
```

### Tabela: itens_entrada
```sql
CREATE TABLE IF NOT EXISTS itens_entrada (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  entrada_id INTEGER NOT NULL,
  produto_id INTEGER NOT NULL,
  lote_id INTEGER,
  preco_compra REAL NOT NULL,
  quantidade INTEGER NOT NULL,
  created_at TEXT DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (entrada_id) REFERENCES entradas(id) ON DELETE CASCADE,
  FOREIGN KEY (produto_id) REFERENCES produtos(id),
  FOREIGN KEY (lote_id) REFERENCES lotes(id)
);
```

### Tabela: vendas
```sql
CREATE TABLE IF NOT EXISTS vendas (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  data_venda TEXT NOT NULL,
  valor_total REAL NOT NULL,
  forma_pagamento TEXT,
  cliente_nome TEXT,
  desconto REAL DEFAULT 0,
  user_id INTEGER,
  created_at TEXT DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id)
);
```

### Tabela: itens_venda
```sql
CREATE TABLE IF NOT EXISTS itens_venda (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  venda_id INTEGER NOT NULL,
  produto_id INTEGER NOT NULL,
  lote_id INTEGER,
  quantidade INTEGER NOT NULL,
  preco_unitario REAL NOT NULL,
  subtotal REAL NOT NULL,
  created_at TEXT DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (venda_id) REFERENCES vendas(id) ON DELETE CASCADE,
  FOREIGN KEY (produto_id) REFERENCES produtos(id),
  FOREIGN KEY (lote_id) REFERENCES lotes(id)
);
```

### Tabela: caixa
```sql
CREATE TABLE IF NOT EXISTS caixa (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  data_abertura TEXT NOT NULL,
  data_fechamento TEXT,
  saldo_inicial REAL NOT NULL,
  saldo_final REAL,
  user_id INTEGER,
  status TEXT DEFAULT 'aberto',
  created_at TEXT DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id)
);
```

### Tabela: movimentacoes_caixa
```sql
CREATE TABLE IF NOT EXISTS movimentacoes_caixa (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  caixa_id INTEGER NOT NULL,
  tipo TEXT NOT NULL,
  valor REAL NOT NULL,
  descricao TEXT,
  venda_id INTEGER,
  created_at TEXT DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (caixa_id) REFERENCES caixa(id),
  FOREIGN KEY (venda_id) REFERENCES vendas(id)
);
```

## 🔧 Implementação

### Arquivo de Configuração
**Localização**: `lib/database/db.ts`

```typescript
import * as SQLite from 'expo-sqlite';

let db: SQLite.SQLiteDatabase;

export async function initDatabase() {
  db = await SQLite.openDatabaseAsync('appvendas.db');
  
  await db.execAsync(`
    PRAGMA journal_mode = WAL;
    PRAGMA foreign_keys = ON;
  `);
  
  // Executar todas as migrations
  await runMigrations();
}

export function getDatabase() {
  if (!db) {
    throw new Error('Database not initialized. Call initDatabase() first.');
  }
  return db;
}
```

### Migrations
**Localização**: `lib/database/migrations/`

- Criar arquivo para cada versão: `001_initial.ts`, `002_add_column.ts`
- Manter histórico de versões em tabela `schema_migrations`

### Exemplo de Query Helper
**Localização**: `lib/database/queries/produtos.ts`

```typescript
import { getDatabase } from '../db';
import type { Produto } from '@/types/produto';

export async function getProdutos(): Promise<Produto[]> {
  const db = getDatabase();
  const result = await db.getAllAsync<Produto>('SELECT * FROM produtos WHERE ativo = 1');
  return result;
}

export async function getProdutoById(id: number): Promise<Produto | null> {
  const db = getDatabase();
  const result = await db.getFirstAsync<Produto>(
    'SELECT * FROM produtos WHERE id = ?',
    [id]
  );
  return result;
}

export async function createProduto(produto: Omit<Produto, 'id'>): Promise<number> {
  const db = getDatabase();
  const result = await db.runAsync(
    'INSERT INTO produtos (nome, preco_compra, preco_venda) VALUES (?, ?, ?)',
    [produto.nome, produto.preco_compra, produto.preco_venda]
  );
  return result.lastInsertRowId;
}
```

## 🎯 Regras de Negócio

### Produtos
- Produto só pode ser inativado, nunca excluído (se houver movimentações)
- Quantidade total é calculada somando lotes ativos
- Sempre ter pelo menos 1 lote ao criar produto

### Lotes
- Alertar quando validade estiver próxima (30 dias)
- FIFO (First In, First Out) para saída de produtos
- Lote com quantidade 0 deve ser inativado

### Entradas
- Atualizar quantidade de produto ao registrar entrada
- Criar lote automaticamente se não existir
- Registrar histórico completo para auditoria

### Vendas
- Validar estoque antes de confirmar venda
- Deduzir quantidade do lote mais antigo (FIFO)
- Registrar movimentação no caixa automaticamente

### Integridade
- Usar transações para operações que afetam múltiplas tabelas
- Sempre validar FK antes de inserir
- Soft delete (usar campo `ativo`) ao invés de DELETE

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/atimilson) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
