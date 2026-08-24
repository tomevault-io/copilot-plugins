## migrations

> SQL migration conventions using Goose for Cerebro's PostgreSQL schema


# Database Migrations

## Tool: Goose

Migrations are managed by **Goose** (Docker Compose profile `migrate`). Use the `migrate-up` / `migrate-down` Makefile targets for local development.

```bash
make migrate-up    # applies all pending migrations
make migrate-down  # rolls back one step
```

## File Naming

```
NNN_descriptive_name.up.sql
NNN_descriptive_name.down.sql
```

- `NNN` is zero-padded sequential (e.g. `004`, `005`).
- Name describes the **intent**, not the mechanism: `004_add_strategy_snapshots` not `004_alter_table`.
- Every `.up.sql` must have a matching `.down.sql` that fully reverses it.

## SQL Conventions

```sql
-- ✅ GOOD — idempotent, explicit types, named constraints
CREATE TABLE IF NOT EXISTS order_intents (
    id          UUID        PRIMARY KEY DEFAULT gen_random_uuid(),
    symbol      TEXT        NOT NULL,
    side        TEXT        NOT NULL CHECK (side IN ('BUY', 'SELL')),
    quantity    NUMERIC(20,8) NOT NULL,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX IF NOT EXISTS idx_order_intents_symbol ON order_intents (symbol);
```

- Use `TIMESTAMPTZ` (never `TIMESTAMP`) for all time columns.
- Use `NUMERIC(20,8)` for prices and quantities — never `FLOAT` or `DOUBLE PRECISION`.
- Always `IF NOT EXISTS` / `IF EXISTS` for idempotency.
- Name all foreign keys and check constraints explicitly.

## Schema Baseline

| Migration | Tables Created |
|---|---|
| `001_initial_schema` | `order_intents`, `trades` |
| `002_agent_tables` | Agent log tables |
| `003_audit_events` | `audit_events` |

Never modify a migration that has already been applied in any environment. Write a new migration instead.

---
> Source: [AzzBAN/cerebro](https://github.com/AzzBAN/cerebro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
