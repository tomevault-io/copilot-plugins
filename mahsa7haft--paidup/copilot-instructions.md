## paidup

> Instructions for Claude Code. This file overrides default behaviour — follow it exactly.

# CLAUDE.md — PaidUp

Instructions for Claude Code. This file overrides default behaviour — follow it exactly.

---

## Run commands

```bash
# Install dependencies
uv sync

# Run locally
PYTHONPATH=src uv run python -m app.main
# → http://localhost:5002

# Run tests
uv run pytest

# Seed R2 card cache for all current MPs
PYTHONPATH=src uv run python -m app.seed_cards

# Seed donor tags from CSV
PYTHONPATH=src uv run python -m app.seed_tags data/tags.csv
```

---

## Architecture

### Request flow

```
Browser POST /lookup
  → cache.get("paidup:lookup:{name}")     return immediately if hit (Redis, 1h TTL)
  → search_mp()                            Parliament Members API → member_id, name, party
  → [parallel, ThreadPoolExecutor(3)]
      → get_interests() + parse_interests() + deduplicate_donors()
      → get_biography() + parse_biography()
      → get_twfy_data()
  → db.apply_donor_tags()                  attach tags from donor_tags table
  → cache.set(result, ttl=1h)
  → cache.set("paidup:interests:{id}", deduped_interests, ttl=1h)
  → return JSON

Browser POST /analyze  ← two-level cache
  → cache.get(...)                         Redis, 24h TTL
  → db.get_analysis(...)                   Postgres analyses table, 28-day TTL
  → analyze() in ai.py                     Claude claude-sonnet-4-6 — only on full miss
      → loads prompt from prompts/{key}_v{n}.txt
  → db.save_analysis(...)
  → cache.set(...)
  → return JSON

GET /card/{member_id}
  → r2.get_card_url(member_id)             R2 CDN hit → redirect (skip generation)
  → _get_deduped_interests(member_id)      Redis hit or fresh Parliament fetch
  → generate_card()                        Pillow 900×500 landscape PNG
  → r2.upload_card(member_id)
  → return PNG (or redirect to CDN URL)

GET /card/{member_id}/mobile
  → r2.get_card_url(member_id, variant="mobile")
  → _get_deduped_interests(member_id)
  → generate_mobile_card()                 Pillow 500×900 portrait PNG
  → r2.upload_card(member_id, variant="mobile")
  → return PNG (or redirect to CDN URL)

GET /card/{member_id}/badges
  → _get_deduped_interests(member_id)
  → get_badge_layout()                     JSON — badge positions for overlay (desktop only)

GET /metrics
  → PrometheusMetrics scrape endpoint      latency histograms, request counters, error rates
```

### Module responsibilities

| File | Responsibility |
|---|---|
| `main.py` | Flask routes only. Orchestrates calls to other modules; no business logic. |
| `parliament.py` | All UK Parliament API calls (Members + Interests). Also owns `deduplicate_donors`. |
| `theyworkforyou.py` | TheyWorkForYou API. Always returns `None` gracefully if key not set. |
| `ai.py` | Anthropic SDK calls. Prompts loaded from disk at call time. |
| `card.py` | Pillow image generation. Landscape (`generate_card`) and portrait mobile (`generate_mobile_card`). |
| `cache.py` | Redis wrapper (L1). Returns `None` / no-ops silently when Redis is unavailable. |
| `database.py` | PostgreSQL layer (L2). Stores AI analyses for 28 days. No-ops when DATABASE_URL is unset. |
| `r2.py` | Cloudflare R2 card image cache. Accepts `variant` param for desktop vs mobile keys. |
| `metrics.py` | Prometheus counter definitions. Import here to avoid circular imports. |
| `text_utils.py` | `normalize_name()` and `best_fuzzy_match()` — shared between parliament.py and database.py. |
| `seed_cards.py` | CLI script: generate + upload all MP cards to R2. Not imported. |
| `seed_tags.py` | CLI script: bulk-load donor_tags from CSV. Not imported. |

---

## Key design decisions

### Parallel /lookup API calls

`search_mp()` runs first (sequential — need `member_id`). Then `get_interests`,
`get_biography`, and `get_twfy_data` run in parallel via `ThreadPoolExecutor(max_workers=3)`.
Both `search_mp` (10s) and `get_interests` (15s) have explicit timeouts — Parliament's API
has been observed hanging indefinitely without them.

### _get_deduped_interests cache

`/card`, `/card/mobile`, and `/card/badges` all need the same deduped interest list.
`_get_deduped_interests(member_id)` checks `paidup:interests:{id}` in Redis first.
`/lookup` writes to this key after computing, so a card request after a search is a Redis hit
(~5ms) instead of a fresh Parliament API call + TF-IDF (~500–800ms).

If you add a new route that needs interests, use `_get_deduped_interests()` — not a bare
`get_interests()` call.

### Two-level cache for /analyze

`/analyze` uses L1 (Redis, 24h) → L2 (Postgres, 28 days) → Claude API.

- **L1 Redis**: fastest — did someone search this MP in the last 24h?
- **L2 Postgres**: persistent across restarts — have we ever analysed this MP this month?
- **Claude**: only reached when both miss

On L2 hit, Redis is repopulated so the next call doesn't touch Postgres.

The `_cached` field in the response shows which layer served it (`"redis"`, `"db"`, or absent).

### Why 28 days for Postgres TTL

Parliament's Register of Members' Financial Interests is updated within 28 days of changes.
After 28 days an analysis could reference stale data so it is discarded and regenerated.

### Prompt versioning

Prompts live in `src/app/prompts/` as `{key}_v{n}.txt`. `ai.py` picks the highest version
at call time. To update a prompt: copy `summary_v1.txt` → `summary_v2.txt`, edit, restart.
Both caches are invalidated automatically (version is part of both the Redis key and the
Postgres WHERE clause).

### Deduplication happens at the lookup level, not in parse_interests

`parse_interests` produces one dict per raw register entry. `deduplicate_donors` is called
after, so deduplicated names flow consistently to the JSON response, card, and AI context.
Always call `deduplicate_donors(parse_interests(...))` together — never `parse_interests` alone.

### Donor classification in card.py

`_classify_donor(name)` returns `(badge_type, logo_domain)`:
- Checks `donor_company_links` DB table first (fast path — Claude Haiku writes here)
- If `logo_domain` is set and isn't `NO_COMPANY` → `"company_logo"`, render with favicon
- If `logo_domain` is `NO_COMPANY` → fall back to `_is_person()` heuristic
- DB miss → call Claude Haiku once, write result, never call again for that name

### Desktop vs mobile cards

- Desktop: `GET /card/{id}` → 900×500 landscape, badge overlay via `/badges`
- Mobile: `GET /card/{id}/mobile` → 500×900 portrait, stacked donor rows, no badge overlay

The frontend (`index.html`) switches between them: `window.innerWidth < 600` → mobile URL,
otherwise desktop. Badge overlay fetch is skipped on mobile (not applicable to list layout).

R2 keys are `cards/{id}_{YYYY-MM}.png` (desktop) and `cards/{id}_{YYYY-MM}_mobile.png`.

### Prometheus metrics

`GET /metrics` is exposed automatically by `prometheus-flask-exporter` with route latency
histograms, request counters, and error rates.

Custom counters in `metrics.py`:
- `paidup_lookup_cache_total{layer="redis|miss"}` — /lookup cache hit rate
- `paidup_analysis_cache_total{layer="redis|db|api"}` — which layer served /analyze

To add a new metric: define it in `metrics.py`, import it where needed.
Never define Prometheus objects inline in route handlers — causes duplicate registration errors.

### Constituency can be an empty string

`mp.get("memberFrom", "")` returns `""` for some MPs (e.g. the Prime Minister).
The `/analyze` validation checks `"interests" not in data`, not that constituency is truthy.
Do not tighten this to a truthiness check.

### TheyWorkForYou is optional

`get_twfy_data()` returns `None` if `THEYWORKFORYOU_API_KEY` is not set.
All callers handle `None` gracefully. The UI hides the TWFY stats section when `twfy` is null.

### Fuzzy donor deduplication

`deduplicate_donors` in `parliament.py` uses TF-IDF cosine similarity at a threshold of **0.82**
to cluster near-identical donor names. The canonical name is whichever variant appeared first
in the sorted-by-value list. Merged variants are stored in an `aliases` field shown in the UI.

Known limitation: legal suffixes (`Ltd`, `Limited`, `plc`) can push similar names below the
threshold. Fix planned: normalise suffixes before comparison.

---

## Programming standards

These apply to every change in this repo. Claude Code must follow them without exception.

### No comments by default

Do not add comments unless the WHY is non-obvious: a hidden constraint, a subtle invariant,
a workaround for a specific bug. Do not comment what the code does — well-named identifiers
do that. Do not reference the current task, ticket, or callers in comments.

### No dead code

Remove functions and parameters when they become unused. Do not leave `_old_` variants,
commented-out blocks, or `# TODO: remove this` markers. If something is unused, delete it.

### No over-engineering

Implement exactly what the task requires. No helper abstractions for hypothetical future use.
No feature flags. No backwards-compatibility shims. Three similar lines is better than a
premature abstraction.

### Error handling only at real boundaries

Do not add try/except for things that cannot fail. Validate at system boundaries (user input,
external APIs). Trust internal code and library guarantees. Do not add fallbacks for
hypothetical errors in internal calls.

### External API calls always have timeouts

All `requests.get/post` calls must include `timeout=`. Parliament's API, Clearbit, Google
Favicons — all of them. A hanging worker thread takes down Railway's single-process Flask.
Reasonable defaults: 5s for fast lookups, 10–15s for Parliament Interests.

### Module boundaries

- `main.py` has no business logic — routes only. Any logic beyond trivial glue goes in a module.
- New Prometheus metrics always go in `metrics.py`, never inline.
- New Parliament API calls always go in `parliament.py`.
- New AI calls always go in `ai.py`.

### Cache keys

All cache keys use `cache.make_key()` to normalise inputs.
Format: `paidup:{entity}:{id_or_param}`.
Existing keys: `paidup:lookup:{name}`, `paidup:interests:{member_id}`, `paidup:analyze:{...}`.

### Tests

Tests live in `tests/`. Run with `uv run pytest`.
When adding a route or changing business logic, add a test.
Tests that require external APIs or PIL rendering are skipped in CI via `pytest.mark.skip`.

---

## Environment variables

| Variable | Required | Notes |
|---|---|---|
| `ANTHROPIC_API_KEY` | Yes (AI features) | claude-sonnet-4-6 for /analyze, claude-haiku for donor classification |
| `THEYWORKFORYOU_API_KEY` | No | Free at theyworkforyou.com/api/key |
| `FLASK_SECRET_KEY` | No (dev default exists) | Set a long random string in production |
| `REDIS_URL` | No | L1 cache; Railway Redis plugin sets this automatically |
| `DATABASE_URL` | No | L2 persistent store; Railway Postgres plugin sets this automatically |
| `PORT` | No | Defaults to 5002; Railway sets this automatically |
| `LANGFUSE_PUBLIC_KEY` | No | Langfuse observability — token usage, cost, latency per Claude call |
| `LANGFUSE_SECRET_KEY` | No | Langfuse observability |
| `LANGFUSE_HOST` | No | Defaults to `https://cloud.langfuse.com`; set for self-hosted |
| `R2_ACCOUNT_ID` | No | Cloudflare account ID — enables R2 card image caching |
| `R2_ACCESS_KEY_ID` | No | R2 API token access key |
| `R2_SECRET_ACCESS_KEY` | No | R2 API token secret |
| `R2_BUCKET_NAME` | No | R2 bucket name (e.g. `paidup`) |
| `R2_PUBLIC_URL` | No | Public bucket URL e.g. `https://pub-xxxx.r2.dev` |

### Grafana monitoring (optional)

To activate Prometheus → Grafana Cloud pipeline:
1. Create a Grafana Cloud free account
2. Deploy `grafana/alloy.river` as a Railway service (image: `grafana/alloy:latest`)
3. Set env vars on the Alloy service: `PAIDUP_HOST`, `PAIDUP_PORT=5002`,
   `GC_PROM_URL`, `GC_PROM_USER`, `GC_API_KEY`
4. Import community dashboard ID 9528 in Grafana Cloud

---
> Source: [mahsa7haft/paidup](https://github.com/mahsa7haft/paidup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
