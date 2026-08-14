## srv-assets-v1

> Repo-specific guidance for OpenCode sessions in `srv-assets-v1` (NestJS file-upload/assets REST API). Only non-obvious facts an agent would likely get wrong.

# AGENTS.md

Repo-specific guidance for OpenCode sessions in `srv-assets-v1` (NestJS file-upload/assets REST API). Only non-obvious facts an agent would likely get wrong.

## Toolchain
- **Use `pnpm`** (lockfile `pnpm-lock.yaml`, `pnpm-workspace.yaml`). README mentions npm in places — ignore that.
- **ESM project**: `"type": "module"` + tsconfig `module: nodenext`. **Every relative import must end in `.js`** (e.g. `import { AppModule } from './app.main/app.module.js'`) even though the source is `.ts`. Omitting `.js` breaks the build. This is the #1 mistake to avoid.
- Local `.env` runs on port **5573**, not 3000.

## Commands
- `pnpm dev` (watch) · `pnpm build` (`nest build` → `dist/`) · `pnpm start:prod` (`node dist/main`).
- `pnpm lint` — ESLint with `--fix` and **type-checked** rules (`projectService`); needs full project context, not isolated files.
- `pnpm test` — Jest, rootDir `src`, pattern `*.spec.ts`. **No specs or `test/` dir exist yet**; the `test:e2e` target points at a nonexistent config. Single test: `pnpm test -- <pattern>`.
- After changes, verify with **`pnpm build` then `pnpm lint`** before considering done.

## Prisma (v7 driver-adapter mode)
- Generator is the Prisma 7 `prisma-client`, output → **`src/generated/prisma/`**. This is **generated + gitignored — never hand-edit**. Run `npx prisma generate` after any `schema.prisma` change. Import the client from `'../generated/prisma/client.js'`.
- The `datasource db` block intentionally has **no `url`** — it's supplied by `prisma.config.ts` (CLI) and the **`@prisma/adapter-mariadb`** (`PrismaMariaDb`) at runtime (`prisma.service.ts`). Don't add a URL back.
- `pnpm-workspace.yaml` `allowBuilds` deliberately disables native builds for `prisma`/`@prisma/engines` (and `sharp`) — correct for driver-adapter mode. Leave it.
- Migrations: `npx prisma migrate dev` (local) / `migrate deploy` (prod). Provider `mysql` against MariaDB.

## Environment
- `src/core/config/envs.ts` Joi-validates env at import time and **throws on failure**. Required keys: `EXTERNAL_API_ASSETS_URL`, **`EXTERNAL_ASSETS_BASE_URL`**, `APP_API_SECRET`, `APP_PORT`, `DATABASE_URL`.
- **`.env.example` is missing `EXTERNAL_ASSETS_BASE_URL`** — copy it from the real `.env` or the app crashes on boot. Always set all five.
- Auth: `ApiKeyGuard` compares the `X-API-Key` header against **`APP_API_SECRET`** (not `API_KEY`, even though both exist in env). Applied per-handler, not globally.
- Global `ValidationPipe`: `whitelist` + `forbidNonWhitelisted` + `transform` (`enableImplicitConversion`). Every DTO property needs a decorator or the request 400s.

## Routing & wiring
- Global prefix `/api` (`main.ts`). Versioning is **literal** in path decorators (`@Post('v1/upload-file')` on `@Controller('file')`) — no NestJS URI versioning. Full paths look like `/api/file/v1/upload-file`.
- Static serving: `pageroot/` at `/` (web UI) and `./upload` at `/uploads` (public downloads, cached 1 day). Paths are computed in `app.module.ts` via `join(__dirname, '../../..', ...)` — check there if relocating.
- `FileModule` is the only feature module; `PrismaModule`, `StorageModule`, `ImageModule` support it.

## Git workflow (git-flow)
- Branches: `main`, `develop`, `feature/featr-NNN`, `release/rls-NNN`.
- `scripts/git-flow-release.sh` finishes a feature → start/finish release → push `main`+`develop`+tags → open next `featr-NNN+1`. Run only on a clean `feature/featr-NNN` branch.

## Gitignored but present locally
Don't expect these in PRs/CI (they're in `.gitignore`): `/scripts`, `/src/generated/prisma`, `/docs`, `/.github`, `/.agents`, `/upload`, `.github/copilot-instructions.md`.

## Local DB scripts
- `node scripts/generate-schema.mjs` dumps live table DDL to `database-objects/schemas/`. It reads **its own env keys** — `DATABASE_HOST`, `DATABASE_PORT`, `DATABASE_NAME`, `DATABASE_USER`, `DATABASE_PASSWORD` (not `DATABASE_URL`) — and needs `mysql2`.

## Dependency updates
- See `docs/comandos-atualizacao-latest.md` for pnpm commands tiered by risk (safe/attention/breaking), and the `.agents/skills/safe-dependency-update` skill. Always bump `prisma` and `@prisma/client` together.

---
> Source: [leonardodavinci2049/srv-assets-v1](https://github.com/leonardodavinci2049/srv-assets-v1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
