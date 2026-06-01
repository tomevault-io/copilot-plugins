## next-music-player

> This is a **Next.js Music Player** — a single full-stack Next.js 16 app (not a monorepo). It uses PostgreSQL via Drizzle ORM, Vercel Blob for file storage, and Tailwind CSS v4 for styling. See `README.md` for feature list and standard commands.

# AGENTS.md

## Cursor Cloud specific instructions

### Overview

This is a **Next.js Music Player** — a single full-stack Next.js 16 app (not a monorepo). It uses PostgreSQL via Drizzle ORM, Vercel Blob for file storage, and Tailwind CSS v4 for styling. See `README.md` for feature list and standard commands.

### Services

| Service | How to start | Notes |
|---------|-------------|-------|
| PostgreSQL | `sudo docker compose up -d` (from repo root) | Uses `docker-compose.yml` with `postgres:16.4-alpine` on port `54322`. Must have Docker running (`sudo dockerd` if not started). |
| Next.js dev server | `pnpm dev` | Runs on `http://localhost:3000` with Turbopack. |

### Environment variables

The `.env` file must contain `POSTGRES_URL` and `BLOB_READ_WRITE_TOKEN`. For local dev with Docker Postgres, use `POSTGRES_URL=postgres://postgres:postgres@localhost:54322/postgres`. The `BLOB_READ_WRITE_TOKEN` is only required for file upload features (playlist cover images, song uploads via seed script); the app starts and serves pages without a valid token.

### Database

- **Migrations**: `pnpm db:migrate` (requires `tsx` installed globally: `npm install -g tsx`)
- **Seed**: `pnpm db:seed` requires `.mp3` files in a `tracks/` directory and a valid `BLOB_READ_WRITE_TOKEN`. For quick dev testing, seed data directly via `psql` into the Docker container.
- **Studio**: `pnpm db:studio` opens Drizzle Studio for database inspection.

### Build & lint

- **Build**: `pnpm build` — runs TypeScript checking and Next.js production build.
- **Lint**: No dedicated lint script is configured. TypeScript type-checking runs as part of `pnpm build`.
- **Dev**: `pnpm dev` — starts the dev server with Turbopack hot reload.

### Gotchas

- The `pnpm db:*` scripts use `npx tsx` which requires `tsx` to be globally installed (`npm install -g tsx`). It is not a project dependency.
- The `pnpm db:setup` script is interactive (prompts for input). Do not run it in non-interactive mode. Instead, create `.env` and `docker-compose.yml` manually.
- `pnpm.onlyBuiltDependencies` in `package.json` must include `esbuild` and `sharp` for their native binaries to build correctly during `pnpm install`.
- Docker requires `fuse-overlayfs` storage driver and `iptables-legacy` in the Cloud Agent VM. Docker daemon must be started with `sudo dockerd &` before running `docker compose up -d`.

---
> Source: [leerob/next-music-player](https://github.com/leerob/next-music-player) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
