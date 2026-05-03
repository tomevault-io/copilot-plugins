## chatterbox

> Chatterbox is a roleplay chat application focused on long-running continuity and reliable LLM behavior (GLM 5 via OpenRouter). The architecture supports iterative upgrades to prompt assembly, state management, and memory systems without destabilizing the app.

# AGENTS.md — Repository Rules (`chatterbox`)

## Project overview

Chatterbox is a roleplay chat application focused on long-running continuity and reliable LLM behavior (GLM 5 via OpenRouter). The architecture supports iterative upgrades to prompt assembly, state management, and memory systems without destabilizing the app.

Single-page "use client" SPA with API routes for backend. Dark-mode only. Targets desktop and iPhone Safari.

## Quick start

```bash
pnpm install                # install all dependencies
pnpm infra:up               # start Postgres (Docker Compose)
pnpm dev                    # Next.js dev with Turbopack (binds 0.0.0.0)
pnpm start                  # runs production app (binds 0.0.0.0)
```

Requires `apps/web/.env` with at minimum `OPENROUTER_API_KEY`. See `apps/web/src/lib/env.ts` for typed env vars and defaults.

Optional env vars:

- `FACT_EXTRACTION_MODEL` - model ID used by digest fact extraction (`/api/chat`) for PH04 compression.

## Build & validate

| Command                        | Scope        | Purpose                                                       |
| ------------------------------ | ------------ | ------------------------------------------------------------- |
| `pnpm dev`                     | apps/web     | Dev server (Turbopack)                                        |
| `pnpm build`                   | apps/web     | Production build                                              |
| `pnpm typecheck`               | all packages | `tsc --noEmit` recursively                                    |
| `pnpm lint`                    | all packages | ESLint recursively                                            |
| `pnpm infra:up` / `infra:down` | infra        | Postgres via Docker Compose                                   |
| `pnpm fetch:providers`         | root script  | Refresh OpenRouter provider order snapshot for model registry |

Tests exist in `@chatterbox/sockets` and `@chatterbox/state-model` (vitest). Run per-package: `pnpm --filter @chatterbox/state-model test`, etc. Full validation relies on `pnpm typecheck` + `pnpm lint` + per-package tests + `pnpm dev` starts successfully.

## Workspace structure

```text
chatterbox/
├── apps/web/                # Next.js 16 app (React 19, App Router)
├── packages/
│   ├── sockets/             # boundary contract types + defaults (leaf)
│   ├── prompt-assembly/     # segmented prompt engine
│   └── state-model/         # entity-centric story state model (leaf)
├── infra/                   # schema.prisma, migrations, docker-compose.yml
├── dev-docs/                # planning docs (gitignored, local-only)
├── prompts/                 # character prompt assets (gitignored, local-only)
├── scripts/                 # ad-hoc utility scripts
├── tsconfig.base.json       # strict shared TS settings
├── tsconfig.json            # project references hub
├── eslint.config.mjs        # root boundary rules
└── pnpm-workspace.yaml      # workspace package map
```

### Key things that are gitignored

- `prompts/` — character-specific system prompts and story states (local-only)
- `.env*` — environment files

## Dependency graph

```
apps/web → @chatterbox/prompt-assembly → @chatterbox/sockets
apps/web → @chatterbox/sockets
apps/web → @chatterbox/state-model
```

`@chatterbox/sockets` is a strict leaf — it cannot import from any other package.
`@chatterbox/state-model` is a strict leaf — it cannot import from any other package.

## Non-negotiable boundaries

### Markdown characters

<!-- markdownlint-disable -->
<!-- cspell:disable -->

Do not use any typographic or "smart" punctuation.

- Forbidden Characters (non-ASCII)
  - En dash: –
  - Em dash: —
  - Curly quotes: “ ” ‘ ’
  - Ellipsis: …
  - Non-breaking spaces
      <!-- cspell:enable -->
    <!-- markdownlint-restore -->

### Package boundaries

- Every cross-package dependency must be declared in `package.json`.
- Import from package root only (`@chatterbox/sockets`), never `src/*` internals.
- ESLint `eslint-plugin-boundaries` enforces the dependency graph. Boundary violations are architecture regressions.

### TypeScript

- Packages are separate TS projects connected via project references.
- `noUncheckedIndexedAccess: true` — all bracket-access returns `T | undefined`.
- **Packages have no build step.** Both shared packages expose `./src/index.ts` directly. The consuming app's bundler (Turbopack) compiles them. No `dist/` folder, no `.js` entry points.

### ESLint limits (warn-level)

- Cyclomatic complexity: max 10
- Function length: max 100 lines (skip blank/comments)
- Aligned with Codacy/Lizard threshold in `.codacy/tools-configs/lizard.yaml`.

## Database (Prisma)

- Schema lives at `infra/schema.prisma` (not the default location).
- `prisma.config.ts` at workspace root maps this; run all `prisma` CLI commands from root.
- Uses the Postgres driver adapter (`@prisma/adapter-pg`), not Prisma's built-in connection.
- Docker Compose: `pgvector/pgvector:pg16` (Postgres 16 + pgvector), default creds `chatterbox/chatterbox`, host port 55432 mapped to container port 5432.
- `globalThis` caching pattern prevents PrismaClient re-instantiation in dev hot-reload.

### Migration workflow

- Never use `prisma migrate dev` - it detects drift from raw-SQL tables (`MessageEmbedding`/pgvector) and resets the database.
- Use `pnpm db:migrate` (`prisma migrate deploy`) to apply pending migrations safely.
- Create migration SQL files manually in `infra/migrations/` with naming convention `YYYYMMDDNNNN_description/migration.sql`.
- Always use deterministic UUIDs in seed data (never `gen_random_uuid()` for seed rows).
- After creating a new migration, run `pnpm db:generate` to sync the Prisma client.
- Use `pnpm db:status` to check which migrations have been applied.
- Only use `pnpm db:reset` when you intentionally want to drop and recreate everything.

## Conventions

### Code style

- UI components: shadcn/ui (New York style), Radix primitives, Tailwind CSS 4, `cn()` utility from `src/lib/utils.ts`.
- Icons: `lucide-react`.
- Hooks live in `src/lib/hooks/`. Each gets its own file with `use-` prefix.

### Module-level mutable state in page.tsx

`liveConfig` in `page.tsx` is a module-scope mutable object read by the chat transport at request time. This is **intentional** — do not refactor it into React state or add it to any dependency array.

### Dev-docs naming

`{type}{number}-{slug}.md` — types: `PL` (plan), `IM` (implementation). Completed docs move to `completed/` with `(COMP)` or `(DEP)` prefix. Docs use YAML frontmatter with Status and Last Updated.

## AGENTS.md governance

- Every `packages/*` and `apps/*` must have its own `AGENTS.md`.
- When a package/app changes, its local `AGENTS.md` must be updated in the same changeset.
- Updates must remove stale guidance and capture new contracts, boundaries, or integration notes.

## Change policy for new systems

1. Define a package-level contract in `@chatterbox/sockets` first.
2. Add a default implementation that preserves current behavior.
3. Integrate via explicit adapter in `apps/web`.
4. Register the new package in ESLint boundary rules (`eslint.config.mjs`).

## Definition of done

All must pass before merging structural changes:

- `pnpm typecheck`
- `pnpm lint`
- `pnpm dev` starts successfully
- No boundary or deep-import violations introduced

---
> Source: [ceponatia/chatterbox](https://github.com/ceponatia/chatterbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
