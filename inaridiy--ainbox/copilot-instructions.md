## ainbox

> pnpm monorepo. Node 20+, pnpm 10+.

# Contributing / agent notes

pnpm monorepo. Node 20+, pnpm 10+.

## Commands

```sh
pnpm install
cp packages/worker/wrangler.jsonc.example packages/worker/wrangler.jsonc  # first time
pnpm build        # shared → worker (types + d.ts) → cli, in order
pnpm typecheck
pnpm test         # vitest unit tests (packages/worker/test)
pnpm --filter @ainbox/worker dev      # wrangler dev (needs resources; see README)
pnpm --filter @ainbox/worker deploy
```

## Layout

- `packages/worker` — Cloudflare Worker (Hono). Email ingest (`src/ingest.ts`), send (`src/outbound.ts`), HTTP API (`src/api/`), Access auth (`src/auth.ts`), AI classification (`src/classify.ts`).
- `packages/cli` — `ainbox` CLI (commander). Talks to the Worker via Hono RPC (`hc<AppType>`); the worker package emits `.d.ts` for this.
- `packages/shared` — API response DTO types.
- `skills/ainbox` — drop-in skill file for AI coding agents that use the CLI.

## Rules

- DB schema lives in `packages/worker/src/db/schema.ts` (Drizzle). Generate migrations with `pnpm --filter @ainbox/worker db:generate`; never hand-write them. Exception: the FTS5 virtual table is a hand-written custom migration, declared for queries in `src/db/fts-schema.ts` (kept out of `schema.ts` so drizzle-kit ignores it).
- Anything batched into D1 must be a Drizzle query builder — raw `db.run(sql)` inside `db.batch()` fails at runtime.
- FTS-indexed text is CJK-segmented via `src/lib/segment.ts`; index writes and queries must use the same segmentation.
- `packages/worker/wrangler.jsonc` is deployment-specific and gitignored; edit `wrangler.jsonc.example` when config shape changes.
- After changing routes in the worker, rebuild before touching the CLI — the CLI's types come from the worker's emitted declarations.

---
> Source: [inaridiy/ainbox](https://github.com/inaridiy/ainbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
