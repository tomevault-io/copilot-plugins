## effect-worker-mono

> Navigation guide for humans and agents. Read this first, then jump to the source files or docs named below. Detailed knowledge lives in those docs — keep this file short.

# AGENTS.md

Navigation guide for humans and agents. Read this first, then jump to the source files or docs named below. Detailed knowledge lives in those docs — keep this file short.

## Universal Rules

1. Run `pnpm check:all` before every PR. It runs lint (oxlint), ast-grep structural scan, and format check.
2. Run `pnpm test` to verify the test suite.
3. Update docs in the same PR when architecture or workflow changes.
4. Cloudflare `env` and `ctx` are request-scoped. Never cache them globally.
5. Put reusable queries in `@repo/db/src/queries/` — do not scatter raw Drizzle in app handlers.
6. Use `fp issue` for all task tracking. See `FP_AGENTS.md`.
7. Effect v4 reference: `~/.local/share/ai-references/effect/v4/LLMS.md`.

## Package Index

| Package | Purpose | Entry |
|---------|---------|-------|
| `@repo/domain` | Branded schemas, domain errors | `packages/domain/src/index.ts` |
| `@repo/db` | Drizzle schema, `PgDrizzle` service, query programs | `packages/db/src/index.ts` |
| `@repo/cloudflare` | Bindings bridge, middleware factories, observability | `packages/cloudflare/src/index.ts` |
| `@repo/contracts` | HTTP + RPC contracts, middleware tags | `packages/contracts/src/index.ts` |

## App Index

| App | Runtime | Entry |
|-----|---------|-------|
| `effect-worker-api` | Cloudflare Worker (HTTP) | `apps/effect-worker-api/src/index.ts` |
| `effect-worker-rpc` | Cloudflare Worker (RPC) | `apps/effect-worker-rpc/src/index.ts` |
| `tanstack-start` | Cloudflare Worker (full-stack) | `apps/tanstack-start/src/server.ts` |

## Docs by Topic

| Topic | Doc |
|-------|-----|
| Architecture, layer responsibilities, boundaries | `docs/architecture.md` |
| Observability, tracing, NDJSON spans | `docs/observability.md` |
| Boundary conventions (adapters, I/O, entry points) | `docs/agents/patterns/boundaries.md` |
| Effect patterns and enforced anti-patterns | `docs/agents/patterns/effect.md` |
| Coding style | `docs/agents/patterns/coding-style.md` |
| Data validation | `docs/agents/patterns/data-validation.md` |
| Testing principles | `docs/agents/patterns/testing-principles.md` |
| End-to-end testing | `docs/agents/patterns/end-to-end-testing.md` |
| Building a new API app | `docs/agents/templates/api.md` |
| Building a new CLI app | `docs/agents/templates/cli.md` |
| Building a new worker app | `docs/agents/templates/worker.md` |
| Validation gate reference (all check commands) | `docs/agents/validation.md` |
| Continuous improvement loop | `docs/agents/harness-engineering.md` |
| Issue tracking workflow | `FP_AGENTS.md` |

## Verification (run before declaring done)

```bash
pnpm check:all   # oxlint + ast-grep structural scan + format check
pnpm test        # vitest test suite
pnpm check       # tsc type check across all packages
pnpm build       # build shared packages in dependency order
```

See `docs/agents/validation.md` for per-package and per-tool breakdowns.

## Troubleshooting

- `env.HYPERDRIVE` undefined → check `wrangler.jsonc` and `CLOUDFLARE_HYPERDRIVE_LOCAL_CONNECTION_STRING_HYPERDRIVE` in `.env`.
- `/api/openapi.json` missing → OpenAPI is registered in `apps/effect-worker-api/src/runtime.ts`, not the entrypoint.
- RPC returns 404 → handler is at `/rpc`; `/health` is the only non-RPC route in `effect-worker-rpc`.
- Traces missing → run `pnpm dev:traced:api` or `pnpm dev:traced:rpc`, inspect `.traces/spans.ndjson`.
- `tanstack-start` server functions can't access shared services → `effect-runtime.ts` uses `Layer.empty`; wire real layers before assuming dependencies are available.
- Docs disagree with code → trust `package.json`, `src/index.ts`, `runtime.ts`, `wrangler.jsonc` over prose. Fix the prose.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sanurb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
