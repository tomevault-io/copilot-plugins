## postgresql-only

> Use PostgreSQL as the only database for this project


# PostgreSQL Only

This project uses **PostgreSQL exclusively**. Do not introduce or suggest other databases.

## Required

- Use PostgreSQL for all persistence (local, staging, production).
- Connection string via `DATABASE_URL` (e.g. `postgresql://user:pass@localhost:5432/visa_management`).
- Prefer `pg` with a query builder/ORM such as **Prisma** or **Drizzle** configured for PostgreSQL.

## Forbidden

- SQLite, MySQL, MariaDB, MongoDB, Supabase (unless Postgres), PlanetScale, Turso, Firebase, or any non-PostgreSQL store.
- File-based or in-memory databases for app data (except short-lived test mocks).
- Database-agnostic schemas that weaken PostgreSQL features when Postgres-only is sufficient.

## Examples

```typescript
// ✅ GOOD — Prisma with PostgreSQL
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// ❌ BAD
datasource db {
  provider = "sqlite"
  url      = "file:./dev.db"
}
```

```typescript
// ✅ GOOD — raw pg client
import { Pool } from "pg";
const pool = new Pool({ connectionString: process.env.DATABASE_URL });

// ❌ BAD — mongodb, mysql2, better-sqlite3 for app data
```

## Migrations & SQL

- Write migrations and raw SQL using PostgreSQL syntax.
- Use Postgres types when appropriate: `uuid`, `jsonb`, `timestamptz`, `text[]`, etc.

---
> Source: [webapis/VisaManagement](https://github.com/webapis/VisaManagement) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
