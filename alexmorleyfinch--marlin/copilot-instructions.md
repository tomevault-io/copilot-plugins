## marlin

> This is a fully vibe coded project with oversight, so this file reflects the current truth. When the humans request contradicts this file, clarify the intent to stray from the definition.

# AGENT.md

This is a fully vibe coded project with oversight, so this file reflects the current truth. When the humans request contradicts this file, clarify the intent to stray from the definition.

Context for Cursor and future agents working in this repo. Human operator guide: `docs/GETTING_STARTED.md` (entities, config, CLI, LM Studio, vLLM/Vast). Policy inventory: `data/README.md`. Schema: `docs/MIGRATIONS.md`. Do not duplicate those step-by-steps here; keep this file to facts that are expensive to infer.

## What this is

Marlin is a **personal, single-user** search index aimed at tens of millions of domains. No auth, no multi-tenancy. Do **not** add `userId` or a domain-ignore table unless asked. Ignore is a boolean on `categories` and `tags`.

v1 discovery is a **domain list file** (ingest) plus **link following after LM** (outbound enqueue). An optional BFS **spider** can also expand from a few seeds into `pending` — not required for normal use. There is no IPv4/ICMP/TLS-SAN scanner.

## Layout

| Path | Owns |
| --- | --- |
| `apps/spider` | **Ingest** CLI (`src/ingest.ts`: domain list → `pending`) + optional **BFS spider** (`src/index.ts`: seed link-follow → `pending`). Spider is not the main crawl path — prefer ingest + fetcher; primary link growth is LM outbound enqueue after `done` |
| `apps/fetcher` | High-concurrency homepage fetch → store extracted text + outbound hosts (no enqueue) |
| `apps/worker` | Claim `ready` pages: near-empty / bot-check → `empty` (no LM), for-sale lander → `parked` (no LM), else one OpenAI-compatible LM call; `src/probe.ts` is the no-DB smoke test |
| `apps/steward` | Spiral detector: SQL-nominate busy apexes → LM sample judge → auto-block in Postgres; same `WORKER_PROFILE` as catalog worker; policy from `STEWARD_POLICY` / `--policy` |
| `apps/api` | Fastify `/api/*` search (incl. country/language, `{ hits, hasMore }`), ignore toggles, `/api/dashboard`, `/api/workers`, `/api/analyze/*` catalog analysis + label merge + steward unblock |
| `apps/web` | Vite + React search UI (query-string filters + load more), `/dashboard`, `/workers`, `/analyze` (overview / labels / platforms / steward), ignore modal |
| `packages/db` | Drizzle schema, SQL migrations, pool, queries, analyze aggregates, label-merge lib, migrate/requeue/flush-queue/merge-labels CLIs |
| `packages/shared` | Hostname normalize, TLD whitelist (`data/tlds.txt`), ICANN apex + subdomain cap, category/language crawl priority, fetch/extract, LLM JSON schema + catalog/steward policy loaders, geo normalize, `pickSiteName`, lm + worker profiles |
| `data/` | Forkable policy — see `data/README.md` |
| `data/domains.sample.txt` | Tiny ingest file for test runs |
| `data/seeds.makers.txt` | Maker / small-web seed hosts (ingest to bias discovery) |
| `data/blocked-apex.txt` | Seed crawler-trap apex denylist (Forumotion, B2B mills) — bootstrapped into `blocked_apexes` |
| `data/allowed-apex.txt` | UGC / platform apexes the steward must never auto-block |
| `data/category-priority.txt` | Per-category crawl/LM queue weights (edit + restart fetcher/worker) |
| `data/language-priority.txt` | Language demotion weights on outbound enqueue (`en` / `mul` / `default`) |
| `data/tlds.txt` | English-oriented last-label TLD whitelist |
| `data/lm-profiles.json` | Named LM connections (`baseUrl` / `model` / `apiKey` / `timeoutMs`). Used by probe/compare via `--lm`, and by worker profiles via `lm` key |
| `data/worker-profiles.json` | Named worker bundles (`lm` key + `concurrency`). Selected by `WORKER_PROFILE` or `npm run worker -- <name>` |
| `data/catalog-policies.json` | Catalog LM policy (prompt, `textChars`, sampling). Selected by `CATALOG_POLICY` or `--policy` on worker / probe / compare |
| `data/steward-policies.json` | Steward spiral-judge policy (prompt, `sampleSummaryChars`, sampling). Selected by `STEWARD_POLICY` or `--policy` on steward |
| `data/label-aliases.txt` | Tag/category spelling aliases — rewrite at `completeDomain`; CLI/UI merge for rows already in DB |

`packages/db` is the only place schema/SQL should live. `packages/shared` is the only place hostname rules, crawl-priority weights, and the LLM **schema** should live — spider/fetcher/worker/steward must not fork copies. Catalog/steward **prompts and sampling** live in `data/*-policies.json`.

Runtime is TypeScript via `tsx` (dev and Docker). Workspace `exports` point at `src/*.ts`.

## Invariants (do not “simplify” away)

- **Fetch and LM are separate processes.** Fetcher saturates the network; LM worker keeps profile `concurrency` in-flight LM calls with no sleep between successes. Do not merge them back into one sequential job — GPU idle time during HTTP is the whole point of the split.
- **LM profiles vs worker profiles vs policies.** Connections live in `data/lm-profiles.json` (`baseUrl` / `model` / `apiKey` / `timeoutMs`). Long-running workers use `data/worker-profiles.json` (`lm` key + `concurrency` only). Catalog behavior (prompt, `textChars`, sampling) lives in `data/catalog-policies.json`; steward judge behavior in `data/steward-policies.json`. `WORKER_PROFILE` / `CATALOG_POLICY` / `STEWARD_POLICY` select defaults; CLI `--profile` (or positional) and `--policy` override. One-shots require an explicit LM key: `npm run probe -- example.com --lm studio-g4-4b` (optional `--policy`); `npm run compare-models -- studio-g4-4b studio-g4-2b`. Loaders: `packages/shared/src/lm-profile.ts`, `worker-profile.ts`, `catalog-policy.ts`, `steward-policy.ts`. Do not scatter `LM_BASE_URL` / `LM_MODEL` / `WORKER_CONCURRENCY` in `.env`.
- **Staging is Postgres, not Redis.** Extracted title/text/url live on `domains.page_*`; discovered link hosts on `outbound_hosts` until LM complete. Wipe `page_title` / `page_text` / `page_url` / `outbound_hosts` on successful `done` (40M × 4KB must not stick around). LM-failed rows **keep** text and outbound hosts so `npm run requeue -- failed` can go back to `ready` without refetching.
- **One LM call per domain** unless `skipLmReason` fires (`packages/shared/src/page-kind.ts`): **pre-LM heuristics** on title+body — near-empty body (`isNearEmptyBody`: <80 chars or <12 words) or bot-check interstitial (Cloudflare “Just a moment…”, etc.) → category `empty`; clear for-sale / registrar copy (`isParkedLander`) → `parked`. No LM, do not invent a site from hostname/title. `parked` is not a bucket for blank pages. Structured JSON schema first, one prompt-only retry, then `failed` (thin stub summaries count as a miss on both attempts). Empty language/place/country do **not**. Prompt/sampling/`textChars`: `data/catalog-policies.json`. Schema + parse: `packages/shared/src/llm.ts`. Geo coerce: `packages/shared/src/geo.ts`. Caller: `apps/worker/src/lm.ts`. Display `name` is `pickSiteName` in `packages/shared/src/name.ts`.
- **Language / place / country** are nullable on `domains`. Pre-migration `done` rows stay null — no TLD/hostname backfill. Country search filter is exact ISO 3166-1 alpha-2 and excludes nulls. Language filter is exact ISO 639-1 (or `mul`) and excludes nulls. Place is listing meta only (UI may stuff it into `q`). These are not ignore-list entities. LM uses `""` for unknown; parse → null. `catalogWithoutLlm` leaves them null.
- **`page_text` is visible body only.** Fetcher stores `extractPage().body`, not description+host. Production LM payload is `buildLlmPageText` with title + body only (`description: ""`) — meta is not staged in Postgres. Probe / compare-models still pass meta via a fresh `extractPage()` so smoke tests can include it.
- **Ignore is search-time only.** Worker still summarizes ecommerce/news/social so categories can be learned, then toggled off in the UI. Explicit category/tag filters override ignore (so `empty` / `parked` stay reachable).
- **Category/tag identity** is the lowercased exact LLM string (`normalizeLabel`), then `data/label-aliases.txt` rewrite at `completeDomain`. No fuzzy merge. File changes apply to new inserts (~30s reload); existing misspellings still need `npm run merge-labels -- --apply` or Analyze UI merge.
- **Queue is Postgres** `FOR UPDATE SKIP LOCKED`: `pending→fetching` (`claimNextFetch`), `ready→summarizing` (`claimNextLm`). Both claim `ORDER BY priority DESC, id ASC`. (There is no `claimNextDomain` — that name is obsolete.)
- **Crawl priority** (`domains.priority`, config `data/category-priority.txt`): seeds ingest at `seed` weight. Fetcher does **not** enqueue outbound hosts. It stores them on `outbound_hosts` until LM classifies the page, then `completeDomain` inserts those hosts at category weight **plus** language adjust from `data/language-priority.txt` (`packages/shared/src/language-priority.ts`): null/unknown always `0`; defaults `en` `0`, `mul` `-10`, other languages `default` `-50`. Additive — e.g. portfolio `40` + `ja` `-50` → `-10`. Existing `pending`/`ready` rows take `GREATEST` if a better source later links to them. Do not hard-skip “bad” categories/languages — negative weight still dequeues, just later. Spider depth-0 seeds also use `seed` weight; deeper spider hops use `default` (outbound fan-out still waits for LM so category weights apply).
- **Subdomain cap** (`MAX_SUBDOMAINS_PER_APEX`, default 100): at most N non-apex hosts per ICANN eTLD+1 (`domains.apex`, `tldts` with `allowPrivateDomains: false` so `alice.tumblr.com` shares `tumblr.com`). Apex itself is always allowed. Overflow is **not stored** — filter outbound before insert (`storeFetchedPage` / `insertQueuedHosts` / spider). `skipped` does not count toward N; `done`/`failed`/in-flight/`pending` do. No deferred shelf. Migrate backfills `apex` then deletes overflow **pending** only on apexes already over the cap.
- **Fetcher backpressure:** `FETCH_MAX_READY` (default 500) counts `ready`+`summarizing`. Fetcher sleeps instead of claiming when at cap so page text does not unbounded-grow ahead of the GPU.
- **Do not store full HTML.** Fetch + truncated text only (`packages/shared/src/page.ts`; LM input budget from catalog policy `textChars`). Manual redirects must `cancel()` unused response bodies — leaving them unread can crash Node via an undici HTTP/1 assert (`installFetchCrashGuards` in fetcher/worker/spider). Each hop (and the initial URL) is checked with `assertSafeFetchUrl`: http(s) only, no userinfo, DNS must resolve to public addresses (no loopback / RFC1918 / link-local / CGNAT / ULA / metadata).
- **API bind defaults to loopback.** Host-run API listens on `127.0.0.1` (`API_HOST` override). Compose sets `API_HOST=0.0.0.0` for published ports. CORS allowlists Vite/Compose origins (`CORS_ORIGINS`).
- **No IPv4 scanning** in v1.
- **`normalizeHost`** strips `www.`, lowercases, rejects IPs/localhost/no-TLD.
- **English TLD whitelist** (`data/tlds.txt`, loader `packages/shared/src/tlds.ts`): last label only. Path override `TLD_FILE`; process-wide replace with `TLD_WHITELIST`.
- **Blocked apexes** (`blocked_apexes` table + seed `data/blocked-apex.txt`): crawler traps (Forumotion farms, B2B vendor microsite hosts, steward-detected hotels/SEO mills). `isIndexableHost` refuses apex + subdomains (file ∪ DB overlay, refreshed ~30s). Startup / migrate deletes unfinished rows. **Keeps `done`.** Not a UGC sample cap — those are link-farm black holes. Steward never auto-blocks `data/allowed-apex.txt` (Tumblr, Neocities, …).
- **Steward** (`apps/steward`): separate from catalog LM. SQL nominates busy apexes (junk category mix / spam-lang mix / hotel-name heuristic) → samples 5 then +5 done hosts → LM `block|keep|unsure` → auto-`blockApex`. Does not claim `ready` rows. Same `WORKER_PROFILE`; judge prompt/sampling from `data/steward-policies.json` (`STEWARD_POLICY` / `--policy`).
- **No non-English language subdomains** (`packages/shared/src/language-subdomain.ts`): `tldts` with `allowPrivateDomains: true` (unlike apex, which uses `false`). Under private suffixes (`blogspot.com`, `github.io`, `tumblr.com`) each host is its own registrable name so UGC accounts like `de.github.io` are not treated as language editions. On public eTLD+1s, every label before the root is checked — skip `fr.wikipedia.org`, `tr.mitsubishielectric.com`, `arz.wikipedia.org`; keep `en.` / `en-us` and apex `wikipedia.org`. `.co.uk` is PSL-safe. Combined gate is `isIndexableHost` (enqueue, spider, fetcher, LM claim). Startup bulk-skip (`skipDisallowedTldQueue`) only covers TLD whitelist; language-subdomain / blocked-apex hosts are skipped per-claim via `hostSkipReason`.
- **Never edit an applied migration.** Next file is after `0008_analyze_indexes.sql`.
- **No discovery edge / link-parent graph in v1.** `outbound_hosts` is wiped on `done`. Analyze Platforms uses apex fan-out + `source` (`list`|`spider`|`link`) proxies only. Do not add a multi-GB edge table without an explicit footprint decision.
- **Analyze UI** (`/analyze`): deep catalog read — Labels (co-occurrence, lexical merge), Platforms (apex quality), Steward (block ledger). Dashboard = ops snapshot; Workers = live pipeline. Merge logic lives in `packages/db/src/label-merge.ts` (CLI + API).
- **LM is an OpenAI-compatible HTTP server.** Local = LM Studio on the host (profile `local` / Compose `docker-g4-4b`). Rented GPU = public vLLM image on Vast (`docs/vast-templates/`, profile `vast-g4-4b-1`) over SSH tunnel. Worker stays on the PC; the GPU box does not touch Postgres.
- **Catalog model (v1):** Gemma 4 E4B (`google/gemma-4-E4B-it` on Vast, `google/gemma-4-e4b` in LM Studio). LM profiles in `data/lm-profiles.json`; catalog policy `v1-simple` has `textChars` **4000** and `max_tokens` **400**; Vast concurrency **32** (≤ vLLM `--max-num-seqs`), server `--max-model-len` **5184**. Throughput / cost notes: `docs/GETTING_STARTED.md` (Path B). Do not cut `textChars` without a quality A/B.

## Data flow

```
domains.txt  --ingest-->  pending (seed priority)
seeds        --spider-->  pending (seed / default by depth; no outbound dump)
pending      --fetcher--> fetching → fetch homepage → extract → page_* (body) + outbound_hosts → ready
ready        --lm worker--> summarizing → empty body / CF challenge → empty (no LM)
                          → for-sale lander → parked (no LM)
                          → else LM → done (page_* + outbound_hosts cleared)
                          → enqueue outbound hosts at category + language crawl priority
                          | failed (page_* + outbound_hosts kept)
UI search    --api-->     done rows, hide ignored category OR any ignored tag (unless that label is in the query)
```

Statuses: `pending` | `fetching` | `ready` | `summarizing` | `done` | `failed` | `skipped`.

Startup reclaim: fetcher maps `fetching`/`processing` → `pending`. LM worker maps `summarizing` → `ready`.

`domain_count` on categories/tags is a counter cache incremented once in `completeDomain` when status is `summarizing` → `done`. Double-complete is a no-op (`aborted: true`) so counts cannot inflate from races. Re-cataloging already-`done` rows is **unsupported** in v1 (would need to decrement the old labels). `npm run merge-labels -- --apply` recalculates counts with `COUNT(*)` for merged labels only. Do not `COUNT(*)` 40M rows for the ignore modal.

## Agent workflow (checks)

After changing code, **check only the packages you touched** — do not run the full monorepo by default.

```bash
npm run check -w @marlin/shared   # example: only shared
npm run check -w @marlin/db       # example: only db
```

Each workspace `check` runs `typecheck` + `lint`, and `test` **only if that package has tests** (today: `shared`, `db`, `worker`). Packages without tests omit `test` from their local `check`.

Touched several packages? Run each `-w` you changed (dependency order if unsure: `shared` → `db` → apps). Full tree (ordered):

```bash
npm run check
```

Root order: shared → db → fetcher → worker → spider → steward → api → web.

## Workflows

Human setup/CLI: `docs/GETTING_STARTED.md`. Short agent reminders below.

**Dev:** Postgres via Compose (host **5433** → container 5432); apps on the host. `npm run dev` = api+web. LM Studio on host. Vite `:5173` → `/api` → `:3000`. Root `.env` via `packages/db/src/env.ts`.

**Safe test order:** LM Studio → `npm run probe -- example.com --lm studio-g4-4b` → migrate → ingest → **`npm run fetcher`** + **`npm run worker`** (two terminals) → UI → ignore modal → spider last.

**Schema change:** edit `packages/db/src/schema.ts` → new SQL in `packages/db/migrations/` → `npm run db:migrate`. Compose `migrate` must stay a dependency of api/fetcher/worker/spider/steward.

**Prod-ish:** `npm run docker:up` (postgres+migrate+api+web). Crawl tools: `npm run docker:up:tools`. Dev Postgres only: `npm run docker:db`. `npm run docker:down` stops containers without deleting volumes.

**Requeue:** `npm run requeue -- failed` → rows with `page_text` become `ready`, others `pending`. No auto-retry loop.

**Flush unfinished crawl:** `npm run flush-queue` deletes `pending`/`fetching`/`ready`/`summarizing`/`failed`/`skipped`, keeps `done`. Full wipe: Compose `down -v` (see `docs/MIGRATIONS.md`).

## LM / fetch pitfalls

- `WORKER_PROFILE` concurrency ≤ LM Studio Parallel (or vLLM max-num-seqs). Parallel N **divides** loaded context. Catalog policy `textChars` default 4000 (do not cut without a quality A/B); `max_tokens` default 400. Do not prompt-only retry context-exceeded errors. Thin summary after the one retry → `failed` (keeps page text). Worker soft-starts at `WORKER_RAMP_START` (8) and adds `WORKER_RAMP_STEP` (8) every `WORKER_RAMP_MS` (30s) up to profile concurrency — avoids cold vLLM prefill OOM; `WORKER_RAMP_MS=0` disables. Rented GPU recipes: `docs/vast-templates/` (docs only).
- `FETCH_CONCURRENCY` default 16 (network). Raising LM concurrency does not require lowering fetch; `FETCH_MAX_READY` is the coupling knob.
- Empty LM queue: poll `WORKER_POLL_MS` (200). After a response, claim immediately — do not add delay on the success path.
- If LM is down, mark `failed` and keep page text + outbound hosts. Ctrl+C mid-summarize → next LM worker start reclaims to `ready`.

## Search pitfalls

- Empty `q` = browse latest `done` (`domains_done_processed_at_idx`). Fuzzy on summary/host/name. Typeahead hits `categories`/`tags` (by `domain_count`), `countries`, and `languages` (distinct codes on `done`). Multiple tags are AND. Country/language filters are exact and skip nulls (old rows).
- UI search state lives in the query string on `/`: `q`, `category` (id), `tags` (comma ids), `country`, `language`. Back/forward and reload restore it. Dashboard category/tag bars and result pills navigate there. API returns `{ hits, hasMore }`; UI paginates with `limit`/`offset` (fetch `limit+1`).
- Hide ignored category **or** any ignored tag, unless the search explicitly filters to that category/tag (then category ignore is also lifted for tag filters, so empty-tagged empty sites show).
- Do not add extra trgm indexes on `domains` casually. Browse/language filters use partial **btrees** in `0007_search_indexes.sql`, not GIN.

## Scale notes

~40M metadata × ~1KB plus trgm. Page text is a **bounded staging buffer** (`FETCH_MAX_READY`), not a permanent corpus. Unique `host` is the dedup key.

## Explicit non-goals (v1)

IPv4/TLS scanning, user accounts, recrawl scheduler, robots.txt beyond UA+delay, storing HTML, Redis, multi-tenant ignore lists, cleaning near-duplicate categories.

## Where to look

- Schema / queue / search: `packages/db/src/schema.ts`, `packages/db/src/queries.ts`
- Migrations: `packages/db/migrations/0001_init.sql` … `0008_analyze_indexes.sql`
- Analyze aggregates / merge: `packages/db/src/analyze.ts`, `packages/db/src/label-merge.ts`
- Web Analyze: `apps/web/src/Analyze*.tsx`
- Language / place / country: `packages/shared/src/geo.ts`, `packages/shared/src/llm.ts` (schema/parse)
- Crawl weights: `data/category-priority.txt`, `data/language-priority.txt`, `packages/shared/src/category-priority.ts`, `packages/shared/src/language-priority.ts`
- TLD whitelist: `data/tlds.txt`, `packages/shared/src/tlds.ts`
- LM profiles: `data/lm-profiles.json`, `packages/shared/src/lm-profile.ts`
- Worker profiles: `data/worker-profiles.json`, `packages/shared/src/worker-profile.ts`
- Catalog policies: `data/catalog-policies.json`, `packages/shared/src/catalog-policy.ts`
- Steward policies: `data/steward-policies.json`, `packages/shared/src/steward-policy.ts`
- Apex / subdomain cap: `packages/shared/src/apex.ts`, `packages/db/src/queries.ts` (`insertQueuedHosts`, `trimApexQueueOverflow`)
- Crawler-trap apex denylist: `packages/shared/src/blocked-apex.ts`, `data/blocked-apex.txt`, `data/allowed-apex.txt`
- Steward spiral judge: `apps/steward`, `packages/shared/src/steward.ts` (JSON schema), `data/steward-policies.json`
- Fetch + extract: `packages/shared/src/page.ts`, `apps/fetcher/src/index.ts`
- Empty / challenge / parked (no LM): `packages/shared/src/page-kind.ts`
- LM loop: `apps/worker/src/index.ts`, `apps/worker/src/lm.ts`
- Operator guide: `docs/GETTING_STARTED.md`; policy inventory: `data/README.md`
- Rented GPU (Vast) templates: `docs/vast-templates/` (not loaded by code)
- Compose profiles: `docker-compose.yml` (`tools`); host Postgres helper: `npm run docker:db`. Compose workers use `DOCKER_WORKER_PROFILE` (default `docker-g4-4b`), not host `WORKER_PROFILE`.
- Env: `.env.example`

---
> Source: [alexmorleyfinch/marlin](https://github.com/alexmorleyfinch/marlin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
