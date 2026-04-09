## litecoin-ordinals-explorer

> @Architect — data layer, Prisma vs Sequelize, revival constraints


# Architect (@Architect)

## Single system of record

- **Prisma** (`backend/prisma/schema.prisma`) is the target for **explorer/indexer** data. Migrations and the generated client must stay aligned.
- **Sequelize** (`backend/app/models/`) is **legacy / transitional**. Do not add new domain models only in Sequelize. New features should land in Prisma or be explicitly dual-written during migration.

## Non-negotiables

- After any `schema.prisma` change: run `npx prisma generate` and fix **all** `PrismaClient` usage (model names are singular camelCase in the client, e.g. `inscription`, `updateProgress`).
- **Do not** assume Sequelize tables match Prisma models (`User`, `Invoice`, `Inscription` differ today). Cross-check both before queries or FKs.
- Read **root `MISSION_CONTROL.md`** before large backend changes; run **`scripts/health-check.sh`** and **`scripts/verify-inscription-gate.sh`** before touching protocol/indexer paths listed there.
- **Chain dependencies:** indexer and inscription HTTP helpers assume a running **ord** backed by a synced (or syncing) **Litecoin** node. Default ops order is **mainnet Core first, then ord** (`MISSION_CONTROL.md`); regtest is the shortcut profile.

## Anti-patterns

- Duplicating the same entity in Prisma and Sequelize with different column sets without a documented bridge.
- Indexer logic that infers “last block” only from `inscription` rows when `UpdateProgress` exists for that job.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IndigoNakamoto)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/IndigoNakamoto)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
