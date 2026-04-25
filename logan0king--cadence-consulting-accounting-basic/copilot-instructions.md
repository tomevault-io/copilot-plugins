## cadence-consulting-accounting-basic

> Source of truth for DB structure (Prisma)


## Rule: Read Prisma schema before any DB decision

**Non-negotiable:** Treat `schema.prisma` as the single source of truth. Do not propose or apply DB changes without reading it first.

### 1) Locate and load the active schema
- Find the nearest **schema.prisma** to the files in context:
  1. `./prisma/schema.prisma`
  2. workspace root `schema.prisma`
  3. the package that declares Prisma scripts in **@package.json**
- Open the located file (e.g., **@prisma/schema.prisma**).  
- Also load if present: **@prisma/migrations/**, **@.env.local**, **@.env**, and **@package.json**.

### 2) Understand the current structure
- Summarize:
  - `datasource` provider and the env var used for `url`
  - `generator` blocks and preview features
  - Models, relations, enums
  - IDs, composite IDs, `@unique`/`@@unique`, indexes, defaults
- If commands are allowed, run safe checks only:
  - `npx prisma format`
  - `npx prisma validate`
  - `npx prisma migrate status`  # read-only

### 3) Decision policy (schema-first)
- All structural changes must be expressed in **schema.prisma** with explicit attributes (`@id`, `@default`, `@updatedAt`, `@relation`, `@@index`, etc.).
- Prefer additive, backward-compatible changes. For renames/drops, plan a migration that preserves data (rename + backfill), not a blind drop.

### 4) Migration rules
- **Dev:** create migrations with  
  `npx prisma migrate dev -n "<short-reason>"`
- **Prod/CI:** apply with  
  `npx prisma migrate deploy`
- **Never** use `prisma db push` on shared/prod DBs. Only consider `db push` on disposable/branch DBs (explicitly requested).

### 5) Post-edit validation
- After edits: `prisma format` → `prisma validate` → `prisma generate`.  
- Ensure generators (e.g., Prisma Client, ERD) complete without warnings.

### 6) Report back
- State the schema path used, provider, pending/created migrations, and a one-paragraph rationale for any change.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/logan0king) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
