## 106-postgres

> How write PostgresQL queries

## Context

* We use Prisma with Postgres
* Migration should be write in Postgres

## Rules

You must always write PostgresQL code when you are in `.sql` files.

PostgresQL use quotes in the tables, here is a valid example :

```sql
INSERT INTO "OrganizationPlan" ("id", "name", "maximumMembers", "createdAt", "updatedAt")
VALUES 
  ('FREE', 'Free Plan', 1, CURRENT_TIMESTAMP, CURRENT_TIMESTAMP),
  ('PRO', 'Pro Plan', 10, CURRENT_TIMESTAMP, CURRENT_TIMESTAMP)
ON CONFLICT ("id") DO NOTHING;
```

- Use valid PostgresQL syntax with guillemet for table and column names.

---
> Source: [cortex225/playerConnect](https://github.com/cortex225/playerConnect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
