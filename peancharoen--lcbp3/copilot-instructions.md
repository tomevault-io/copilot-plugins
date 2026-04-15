## lcbp3

> | ❌ Forbidden                                    | ✅ Correct Approach                             |


# Forbidden Actions

## ❌ Never Do This

| ❌ Forbidden                                    | ✅ Correct Approach                             |
| ----------------------------------------------- | ----------------------------------------------- |
| SQL Triggers for business logic                 | NestJS Service methods                          |
| `.env` files in production                      | `docker-compose.yml` environment section        |
| TypeORM migration files                         | Edit schema SQL directly (ADR-009)              |
| Inventing table/column names                    | Verify against `schema-02-tables.sql`           |
| `any` TypeScript type                           | Proper types / generics                         |
| `console.log` in committed code                 | NestJS Logger (backend) / remove (frontend)     |
| `req: any` in controllers                       | `RequestWithUser` typed interface               |
| `parseInt()` on UUID values                     | Use UUID string directly (ADR-019)              |
| Exposing INT PK in API responses                | UUIDv7 (ADR-019)                                |
| AI accessing DB/storage directly                | AI → DMS API → DB (ADR-018)                     |
| Direct file operations bypassing StorageService | `StorageService` for all file moves             |
| Inline email/notification sending               | BullMQ queue job                                |
| Deploying without Release Gates                 | Complete `04-08-release-management-policy.md`   |
| AI direct cloud API calls                       | On-premises Ollama only (ADR-018)               |
| AI outputs without human validation             | Human-in-the-loop validation required (ADR-020) |

## Schema Changes (ADR-009)

- **NO TypeORM migrations** — edit SQL schema directly
- Always check `specs/03-Data-and-Storage/lcbp3-v1.8.0-schema-02-tables.sql` before writing queries
- Update Data Dictionary when changing fields

## UUID Handling

See `01-adr-019-uuid.md` for complete UUID rules.

Quick reminder:

- ❌ `parseInt(uuid)` → NEVER
- ❌ `Number(uuid)` → NEVER
- ✅ Use UUID string directly

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/peancharoen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
