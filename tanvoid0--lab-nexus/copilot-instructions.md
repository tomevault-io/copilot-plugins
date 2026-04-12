## lab-nexus

> Fresh reseed after Prisma schema or seed data changes (local dev)


# Database: fresh reseed when entities change

When you **add a new Prisma model** (or materially **change** an existing one) in `prisma/schema.prisma`, or you change what **`pnpm db:seed`** must populate (including `prisma/seed.ts`, `prisma/inventory-seed.ts`, and related seed helpers):

1. **Keep wipes complete:** extend **`prisma/clear-database.ts`** so `clearAllData()` deletes every **new** collection (respect **foreign-key / referential order**: children before parents, same style as existing `deleteMany` calls). CI and `pnpm db:reset` rely on a full clear before reseeding.
2. **Refresh local dev data:** after `pnpm db:push` (or alongside it), run **`pnpm db:reset`** — runs **`pnpm db:clear`**, **`prisma db push`**, and **`pnpm db:seed`** — so the database matches the new schema and seed **from a clean slate**. Prefer this over **`pnpm db:seed` alone** when old documents could be incompatible or stale.
3. **Production / shared DBs:** **`db:clear` refuses `NODE_ENV=production`** (and `VERCEL_ENV=production`). Do not use reset workflows against production; use normal migration / data backfill procedures instead.

**Quick reference:** `pnpm db:reset` · `pnpm db:clear` · `pnpm db:seed`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tanvoid0)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tanvoid0)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
