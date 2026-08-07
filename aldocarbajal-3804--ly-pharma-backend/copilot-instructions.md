## ly-pharma-backend

> - **Runtime**: Express 5 + TypeScript 6 + Prisma 7 (PostgreSQL via Supabase)

# Ly-Pharma-backend

## Stack
- **Runtime**: Express 5 + TypeScript 6 + Prisma 7 (PostgreSQL via Supabase)
- **Package manager**: `pnpm` (v10, corepack-managed)
- **Entry point**: `src/index.ts`

## Commands
| Command | Action |
|---|---|
| `pnpm dev` | Dev server via nodemon + ts-node (watches `src/`) |
| `pnpm start` | Run compiled `dist/index.js` |
| `pnpm lint` | ESLint on `src/**/*.ts` |

No test, typecheck, or build scripts exist yet — add them before relying on those steps.

## Database (Prisma + Supabase)
- **Schema**: `prisma/schema.prisma`
- **Prisma config**: `prisma.config.ts` — uses `defineConfig` (Prisma 7). Run all `prisma` CLI commands through it:
  ```
  pnpm prisma generate --config prisma.config.ts
  pnpm prisma db seed --config prisma.config.ts
  pnpm prisma migrate dev --config prisma.config.ts
  ```
- **Two connection strings** in `.env` — **critical distinction**:
  - `DATABASE_URL` (port 6543, `pgbouncer=true`) — **runtime** connections (used by `PrismaPg` adapter in app code and seed)
  - `DIRECT_URL` (port 5432) — **CLI** connections (set in `prisma.config.ts` `datasource.url`; session-mode, no PgBouncer)
- **Prisma 7 removes `url`/`directUrl` from `schema.prisma`** — they go in `prisma.config.ts` only. `directUrl` is removed entirely; use a single `url` pointing to `DIRECT_URL` in the config.
- **Import PrismaClient from `@prisma/client`** (never from `@prisma/client/extension` — that export stays as a stub even after generation).
- **Seed**: `prisma/seed.ts` — uses `PrismaPg` adapter with `DATABASE_URL` (runtime connection via pooler). Set `migrations.seed` in config to `"tsx prisma/seed.ts"`.
- No migrations directory yet — run `prisma migrate dev --config prisma.config.ts` to create one.

## Project Structure
```
├── src/
│   ├── index.ts          # Express app entry point
│   └── generated/prisma/ # Prisma client (gitignored, not created yet)
├── prisma/
│   ├── schema.prisma     # DB schema (8 models)
│   ├── seed.ts           # Seed script
│   └── migrations/       # Not created yet
├── tsconfig.json         # ES6/CommonJS, strict, rootDir: src, outDir: dist
└── package.json
```

## Key Notes
- **Express 5** — error handlers use 4 params; async error handling conventions may differ from Express 4.
- **TypeScript 6** — verify compatibility with ts-node (v10.9.2) and tsx (v4.22.3) bundled.
- **ESLint 10** — review config if `.eslintrc.*` is added; currently no ESLint config file exists.
- **Models**: `empleados`, `usuarios`, `roles`, `clientes`, `ventas`, `detalle_venta`, `productos`, `proveedores`, `tipos_medicamento` — table names are lowercase Spanish plural.
- `.env` contains live Supabase credentials — never commit.
- No CI/CD, no pre-commit hooks, no test framework configured.

---
> Source: [AldoCarbajal-3804/Ly-Pharma-backend](https://github.com/AldoCarbajal-3804/Ly-Pharma-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
