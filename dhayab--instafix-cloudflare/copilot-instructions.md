## instafix-cloudflare

> Orientation for AI coding agents working on this repo.

# AGENTS.md

Orientation for AI coding agents working on this repo.

## What this is

A Cloudflare Worker that returns rich OpenGraph / Twitter Card HTML for Instagram URLs, so chat apps (Discord, Telegram, etc.) unfurl them as proper previews. TypeScript, Hono, Photon WASM for image compositing.

Read [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) and [docs/SPEC.md](docs/SPEC.md) before making non-trivial changes.

## Commands

```bash
npm run dev            # wrangler dev on localhost:8787
npm run typecheck      # tsc --noEmit
npm run lint           # oxlint src/
npm run lint:fix       # oxlint --fix src/
npm run format         # oxfmt --write
npm run format:check   # oxfmt --check
npm run test           # vitest run  (no tests yet)
```

**Before claiming work is complete, run all three: `typecheck`, `lint`, `format:check`.** They are fast (sub-second for lint/format, ~2s for typecheck).

## Layout

```
src/
├── index.ts          # Hono app: routes + trailing-slash rewrite middleware
├── env.ts            # Binding types
├── handlers/         # One file per URL shape; HTML responses and CDN redirects
├── scraper/          # oembed + Browser Rendering + KV merge
├── grid/             # Photon WASM layout, composite, and play-icon overlay
├── views/            # OG/Twitter meta-tag HTML template
└── utils/            # Bot detection, HTML escape, shortcode, share-URL unwrap
docs/                 # ARCHITECTURE.md, SPEC.md
terraform/            # KV namespace + R2 bucket provisioning
wrangler.toml
```

## Code style

- TypeScript strict, `noUncheckedIndexedAccess` on. Array lookups in tight loops typically need `!` assertions — used deliberately.
- No comments that narrate what code does; only comments that capture _why_ or a non-obvious constraint.
- No new library unless it meaningfully simplifies. Prefer native `fetch`, `URL`, `Response`, `HTMLRewriter`.

## Key invariants (read these before editing)

1. **Bump the KV key prefix** (`post:v4:…` → `post:v5:…` in [src/scraper/index.ts](src/scraper/index.ts)) any time `InstaData` or `Media` shape changes. Old entries won't have the new fields; bumping the key makes stale entries key-miss immediately instead of silently producing broken embeds.
2. **Don't use Hono's `trimTrailingSlash()`** middleware — it emits 301. The trailing-slash handler in [src/index.ts](src/index.ts) rewrites internally and re-dispatches; chat scrapers silently drop previews on 3xx.
3. **Always-BR on `/p/` and `/reel/`** even though oembed is cheaper, because oembed can't signal "this is a carousel" or give us video URLs. Cache amortises the cost.
4. **Video CDN URLs expire** (signed with `oe=` timestamp ~6 h out). The 24 h KV cache of `InstaData` means video `URL`s can go stale before the metadata does — `/videos/:id/:n` 302 will then 403 on the CDN side. Accept this; a cache miss re-fetches.
5. **Photon memory is tight.** Free `PhotonImage` instances immediately after use. Never hold more than one decoded full-size source in RAM at a time.
6. **Bot detection is substring-based** and case-insensitive. Adding a UA to [src/utils/bot.ts](src/utils/bot.ts) lets that scraper reach the embed HTML; removing one sends it to the 302 redirect.
7. **The lifecycle logging middleware in [src/index.ts](src/index.ts) MUST remain registered after the trailing-slash rewriter.** The rewriter recursively calls `app.fetch(...)` on the cleaned URL; if logging ran before it, every trailing-slash request would emit two `request.start` / `request.done` pairs. See [docs/OBSERVABILITY.md](docs/OBSERVABILITY.md) for the full log schema.

## Upstream services

- `api.instagram.com/api/v1/oembed` with `X-Ig-App-Id: 936619743392459` — unauthenticated, generous, used for every request
- Cloudflare Browser Rendering REST `/content` — `CF_ACCOUNT_ID` + `CF_BROWSER_API_TOKEN` secrets required; 10 min/day on Workers Free, 10 h/month on Paid. Missing secrets → scraper silently falls back to oembed-only. Per-day call cap in [src/scraper/br-cap.ts](src/scraper/br-cap.ts) (`BR_DAILY_CAP` var, default 300); counter is KV-backed and shares the `POSTS_CACHE` namespace with prefix `br:usage:`
- `scontent.cdninstagram.com` and siblings — for actual media bytes

## Cloudflare resource naming

Convention: every shared-namespace resource (KV namespaces, R2 buckets) is prefixed `instafix-`. The Worker-local binding variable stays short.

| Binding           | Resource name                         |
| ----------------- | ------------------------------------- |
| `env.POSTS_CACHE` | `instafix-posts-cache` (KV namespace) |
| `env.GRIDS`       | `instafix-grids` (R2 bucket)          |

Provisioning is handled by Terraform in [terraform/](terraform/). The KV id is committed into [wrangler.toml](wrangler.toml) (public identifier, not sensitive) so `wrangler deploy` works with no extra tooling. If Terraform ever recreates the namespace, update that one line.

## Logging

See [docs/OBSERVABILITY.md](docs/OBSERVABILITY.md) for the full event vocabulary. Three rules keep log coverage complete as the code evolves:

1. **Every handler must call `c.set('metadata', { handler, outcome, ... })` before returning.** The lifecycle middleware reads this to emit `request.done`. Default is `{ outcome: 'unhandled_error' }` set at middleware entry, so a thrown handler still produces a meaningful terminal event — but any explicit return path should refine it.
2. **New helpers that emit logs take `reqId: string` as an explicit parameter.** No AsyncLocalStorage; the correlation ID travels through signatures so callers are forced to decide what to pass.
3. **Failure events include `reason` and, when available, `stack`.** For `error`-level events (compose failures, unhandled errors), use the `logError` helper in [src/utils/log.ts](src/utils/log.ts). For `warn`-level recoverable failures, use inline `try/catch` with manual `reason`/`stack` extraction — see [src/scraper/oembed.ts](src/scraper/oembed.ts) for the pattern.

The `LOG_LEVEL` env var (`error` | `warn` | `info`, default `info`) gates emission at runtime. It is the kill switch when log volume becomes an incident.

## Common tasks

### Add a new post URL shape

Add a route in [src/index.ts](src/index.ts). If the shape needs a different `kind`, extend `PostKind` in [src/scraper/index.ts](src/scraper/index.ts) and `kindFromPath()` in [src/handlers/embed.ts](src/handlers/embed.ts).

### Change what metadata is extracted

Both scraper sources are in [src/scraper/](src/scraper/). oembed is JSON-over-GET; BR parses the rendered HTML for `carousel_media` or `video_versions`. Both merge in [src/scraper/index.ts](src/scraper/index.ts). If you change the `Media` shape, bump the KV key version (invariant 1).

### Change the embed HTML output

Only [src/views/embed.ts](src/views/embed.ts). All meta-tag emission is conditional on the `ViewsData` fields set by [src/handlers/embed.ts](src/handlers/embed.ts).

### Change the grid composite

Layout math is in [src/grid/layout.ts](src/grid/layout.ts) (pure). Photon pipeline is in [src/grid/compose.ts](src/grid/compose.ts). The handler ([src/handlers/grid.ts](src/handlers/grid.ts)) wires fetch → layout → compose → R2 cache.

## Testing

Vitest + `@cloudflare/vitest-pool-workers` are installed but no suite yet. When adding tests, prefer fixture-based: record a real Instagram oembed response / BR rendered HTML into `test/fixtures/*`, mock `fetch` to replay.

## Things deliberately NOT done

- No `trimTrailingSlash` from Hono (see invariant 2)
- No logging library (Worker logs go to `wrangler dev` stdout or Workers observability)
- No shared constants file for the IG headers — they're few, inlined where used
- No global in-memory LRU for posts (the in-isolate `Map` singleflight is only for dedupe; KV is the canonical cache)

## Documentation

- `docs/ARCHITECTURE.md` — Worker topology, request flows, scraper strategy, storage layers, performance, dependencies
- `docs/SPEC.md` — Route contract, bindings, secrets, data model, configuration constants, known limitations

**IMPORTANT:** Before committing structural changes (new routes, scraper sources, storage changes, new bindings or secrets, dependency changes, CI/workflow changes), you MUST check if `AGENTS.md`, `README.md`, and relevant files in `docs/` need updating. Prompt the contributor to update them if they are affected. This step is mandatory and must not be skipped.

---
> Source: [dhayab/instafix-cloudflare](https://github.com/dhayab/instafix-cloudflare) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
