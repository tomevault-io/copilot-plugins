## x-rank

> handles, then edit `roster` or run `bun run config -- set <handles>`. Only

# x-rank

Self-hostable X (Twitter) leaderboard prototype. Picks a roster of accounts,
fetches their recent posts via the X API v2, persists snapshots in SQLite, and
renders range-aware standings + per-metric leaderboards in a React + Effect Atom
UI. The production path is static: refresh/export locally or in CI, build `dist/`,
and deploy it to any CDN.

## Stack

- **Runtime / build:** Bun + Vite. `bun run dev` launches fake-mode Vite only.
  `bun run dev:static` previews exported snapshots. `bun run dev:live` launches
  Vite plus the API server. `bun run build` typechecks both projects and bundles
  the static client. `bun run publish` refreshes, exports static snapshots, and
  deploys to Vercel.
- **Effect v4 (beta).** The published types lag. If you have a local clone
  of [effect-smol](https://github.com/Effect-TS/effect/tree/main/packages/effect)
  available (commonly at `~/code/open-source/effect-smol`), check that for
  current API shapes before guessing.
- **HTTP:** End-to-end Effect HTTP. The wire contract is one `HttpApi.make(...)`
  spec in `src/api.ts`. The server implements it via `HttpApiBuilder` running
  on `BunHttpServer`. The client consumes it via `AtomHttpApi.Service`, so
  `snapshotAtom` is a typed Atom generated directly from the spec.
- **Persistence:** `@effect/sql-sqlite-bun` over local SQLite. The schema and
  query helpers live in `server/db.ts`. Writes during refresh use
  `db.withTransaction(...)` to coalesce fsyncs.
- **Rate limiting:** `HttpClient.withRateLimiter` with the in-memory
  `RateLimiterStore`. Honors `Retry-After` automatically and re-consumes the
  bucket on 429.

## Layout

```
src/
  api.ts                  HttpApi spec + Schema-defined wire types (shared)
  xrank-config.ts         typed user config helper (`defineXRankConfig`)
  data.ts                 Mock fake-mode data; re-exports types from api.ts
  atoms.ts                Effect Atom graph (rangeAtom, snapshotAtom, dashboardAtom)
                          via AtomHttpApi.Service
  model.ts                Dashboard projection (rankings, summary, formatters)
  App.tsx                 React root; consumes dashboardAtom via useAtomSuspense
server/
  index.ts                BunHttpServer + HttpApiBuilder.layer + Layer.launch
  config.ts               env + normalized config loaded from xrank.config.ts
  db.ts                   SQLite schema + query helpers
  aggregate.ts            buildSnapshot: one batched tweet query, in-JS partition
  refresh.ts              refreshAll Effect; Effect.forEach concurrency 3,
                          db.withTransaction per account
  x/
    XApi.ts               Effect service over HttpClient with rate limit + retry
    Schemas.ts            X API response Schemas with default-key fills
scripts/
  config.ts               agent-friendly roster editor (`bun run config -- ...`)
  doctor.ts               setup validator, safe before refresh
  install-local-scheduler.ts  local launchd/cron publisher installer
xrank.config.ts           user-editable roster, title, and schedule defaults
```

```sh
# fake mode — uses mocked accounts/trend, no network/server
bun run dev

# static self-host path
cp .env.example .env             # set X_BEARER_TOKEN, never commit it
bun run refresh                  # writes ./data/snapshots.db
bun run export                   # writes public/snapshot.json + snapshots/*
bun run dev:static               # previews exported snapshots locally
bun run build                    # dist/ can go to any static CDN

# live local mode — requires X API pay-per-use/read access
export X_BEARER_TOKEN=...        # bearer starts with AAAA, not xai-
bun run dev:live                 # client points at /api/snapshot
bun run refresh                  # direct CLI refresh; server not required

# checks
bun run typecheck
bun run build
```

`xrank.config.ts` is the self-hoster setup surface. Agents should ask for X
handles, then edit `roster` or run `bun run config -- set <handles>`. Only
`handle` is required; `team` and `color` are optional. Run `bun run doctor`
before the first refresh.

`server/index.ts` self-launches via `BunRuntime.runMain(Layer.launch(ApiLive))`.
SIGINT/SIGTERM teardown is handled by the runtime's scope. Without
`X_BEARER_TOKEN`, the server still starts and `/api/health` works, but live
refreshes fail until the token is set. When data already exists, `/api/snapshot`
returns the latest snapshot and can fork a cached refresh; the refresh cache TTL
defaults to one hour via `REFRESH_INTERVAL` (for example, `1 hour`). The
background refresh daemon is off unless `ENABLE_REFRESH_DAEMON=true` is set.

For self-host setup, prefer the static workflow from `README.md`: configure
`xrank.config.ts`, set `.env`, run `bun run doctor`, then `bun run refresh`,
`bun run export`, and `bun run build`. Production should not call the X API at
request time unless the user explicitly wants a live server and accepts the
cost/rate-limit tradeoff.

For deploys, Vercel uses `bun run publish`. Cloudflare Pages should deploy the
static `dist/` directory, e.g. `bunx wrangler pages deploy dist --project-name
x-rank`, with no Worker/D1 requirement.

## Conventions

- **Use `bun`** (not pnpm/node), `bunx` (not npx), `bun eval` (not `node -e`).
- **No test script currently exists.** Use `bun run typecheck` and
  `bun run build` for verification unless tests are added later. If vitest is
  added, run it through `bun run test`, never `bun test`.
- **Effect first.** Prefer Effect primitives — `Stream.paginate`, `Effect.cachedInvalidateWithTTL`,
  `Schema.withDecodingDefaultKey`, `HttpRouter.provideRequest` — over hand-rolled
  loops, ad-hoc caches, or manual `fetch`.
- **`HttpApi` is the contract.** Both server and client read from `src/api.ts`.
  Don't add a new endpoint by editing handlers and clients in parallel; add it
  to the spec first.
- **`Schema` for parsing at boundaries.** ISO strings → `DateTime.Utc` via
  `Schema.DateTimeUtcFromString`. Optional metric counts → numbers via
  `Schema.withDecodingDefaultKey(Effect.succeed(0))`.
- **Comments only when WHY is non-obvious.** Don't narrate WHAT (well-named
  identifiers do that), don't reference tasks/PRs.
- **Commits.** Never include AI attribution, "Generated with Claude Code", or
  Co-Authored-By trailers for AI assistants. Author your own commits.

## Known limits

- Trend chart is hardcoded to last 7 days regardless of `range`. Extend by
  bucketing weekly when range > 7d.
- `previousGrowth` needs ≥3 follower-history samples per account. Run refresh
  enough times before it's meaningful.
- `topPost` URL falls back to `https://x.com/{handle}/status/{id}` if the
  refresh job doesn't capture a canonical URL.
- No write serialization across `Effect.forEach(..., concurrency: 3)` — SQLite
  is single-writer; the WAL handles it but contention is possible at scale.

---
> Source: [kitlangton/x-rank](https://github.com/kitlangton/x-rank) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
