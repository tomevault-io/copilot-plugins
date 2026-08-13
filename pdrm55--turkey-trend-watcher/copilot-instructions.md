## turkey-trend-watcher

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TrendiaTR is a live Turkish news aggregation and trend-scoring platform running at **trendiatr.com**. It ingests from 50+ RSS feeds, 37 Telegram channels, and X (Twitter) trends, clusters semantically related articles using vector embeddings, and scores clusters with a proprietary TPS metric. The system is fully Dockerized and **always running in production** — treat every DB/infrastructure change with care.

## Common Commands

```bash
# Start infrastructure + web (no workers)
sudo docker compose up -d

# Start everything including workers
sudo docker compose --profile workers up -d

# Rebuild images. ALWAYS pass --profile workers: without it, compose builds only
# api_server, dashboard and db_init and silently skips the other 10 services —
# no error, no warning. They stay on the old image while the built three move on.
sudo docker compose --profile workers build
sudo docker compose --profile workers up -d

# Restart the API server (also runs DB migrations via init_db())
sudo docker compose restart api_server

# Restart a specific worker
sudo docker compose restart ttw_gravity   # gravity_worker
sudo docker compose restart ttw_summarizer

# View logs
sudo docker compose logs api_server --tail=50 -f
sudo docker compose logs ttw_gravity --tail=30

# Tests that import app modules run INSIDE a container. The host has pytest but
# none of the app dependencies, so a host run dies on `import sqlalchemy`.
sudo docker exec ttw_api python3 -m pytest tests/test_b2b_api.py -v
sudo docker exec ttw_gravity python3 -m pytest tests/test_scoring_queue_atomicity.py -v

# tests/test_gravity_pagination.py imports nothing from app/ and runs on the host:
python3 -m pytest tests/test_gravity_pagination.py -v

# Run a single test class
sudo docker exec ttw_api python3 -m pytest tests/test_b2b_api.py::TestAuthentication -v

# pytest ships via requirements.txt, so it is only present after an image build:
#   sudo docker compose build api_server && sudo docker compose up -d api_server
# tests/test_summary_analysis_filter.py also runs without pytest:
sudo docker exec ttw_summarizer python3 tests/test_summary_analysis_filter.py

# Syntax-check Python files without running them
python3 -c "import ast; ast.parse(open('app/api/routes.py').read()); print('OK')"
```

## Images & Rebuilds

All 13 services build from the **same `Dockerfile` and the same context** — the
images differ only in tag. So the first service built pays the full cost and the
rest are cache hits (~40s for the other 10).

Application code is bind-mounted, so a code change needs only `git pull` +
`restart`. A rebuild is required only when `requirements.txt` changes.

**`requirements.txt` changes are expensive.** The Dockerfile does
`COPY requirements.txt` before `RUN pip install`, so any edit invalidates that
layer and torch + sentence-transformers + chromadb reinstall from scratch — a
~26 GB layer and several minutes. Check `df -h /` first: the server sits around
83-88 % full, and a build that runs the disk out takes production down with it.
`sudo docker builder prune -f` frees cache but makes the next rebuild pay the
full torch download again.

## Database Migrations

**Schema changes go in `app/database/models.py` — two places:**
1. Add the `Column(...)` to the ORM class
2. Add a migration block in `init_db()` using the existing pattern:
   ```python
   if 'new_column' not in trend_columns:
       conn.execute(text("ALTER TABLE trends ADD COLUMN new_column TYPE"))
   ```

`init_db()` is called automatically at `api_server` startup. Only adds columns, never drops data. After adding a migration, `sudo docker compose restart api_server` applies it.

## Architecture

### Request → Response Path (Web)

```
Browser → Nginx → ttw_api (Flask/Gunicorn 4 workers)
              → web_server.py::create_app()
              → api_bp (routes.py)    — HTML pages + JSON utility endpoints
              → api_v1_bp (api_v1.py) — B2B REST API (/api/v1/)
              → api_admin_b2b_bp      — API client management
```

### Ingestion → Scoring Pipeline

```
rss_fetcher.py / telegram_bot.py / social_worker.py
    → text_utils.clean_text()
    → classifier.fast_classify()
    → ai_engine.process_news()         ← embeds text, queries ChromaDB
          → auto-merge OR ask Ollama (Qwen 2.5:1.5b)
    → DB write (Trend + RawNews + TrendArrivals)
    → scoring_queue.enqueue(trend_id)  ← Redis priority queue

gravity_worker.py (every 5s)
    → TPSCalculator.run_tps_cycle()
    → threshold checks → alert_service or publish_to_channel()
```

### Key Modules

| File | Role |
|------|------|
| `app/core/ai_engine.py` | Central AI pipeline: embed → ChromaDB search → Ollama verify |
| `app/core/scoring.py` | `TPSCalculator` — V/E/S/N signals + criticality boost |
| `app/core/translation.py` | Turkish→Persian translation: Redis → DB → Gemini |
| `app/api/routes.py` | All Flask routes (~2300 lines); contains `invalidate_trend_caches()` |
| `app/database/models.py` | All SQLAlchemy models + `init_db()` migration runner |
| `app/workers/summarizer.py` | Gemini summarization; logs to `ai_monitor_data.csv` |
| `app/workers/gravity_worker.py` | Scoring loop, decay loop, cleanup loop |

### Service Names vs Container Names

Service names (used in `docker compose` commands) differ from container names:
| Service name (docker compose) | Container name |
|-------------------------------|----------------|
| `api_server` | `ttw_api` |
| `db_init` | `ttw_init` |
| `gravity_worker` | `ttw_gravity` |
| `rss_worker` | `ttw_rss` |
| `summarizer` | `ttw_summarizer` |
| `telegram_worker` | `ttw_telegram` |
| `social_worker` | `ttw_social` |
| `merge_worker` | `ttw_merge` |
| `x_worker` | `ttw_x_worker` |
| `market_worker` | `ttw_market` |
| `telegram_bot_worker` | `ttw_interactive_bot` |

Workers require `--profile workers` to start.

## Critical Patterns

### DB Sessions in Routes
Routes use **manual session lifecycle** — not context managers:
```python
db = SessionLocal()
try:
    ...
finally:
    db.close()
```
`translation.py` is the exception: it opens its **own internal sessions** for reads/writes so it never shares or rolls back the caller's session.

### Cache Invalidation
Always call `invalidate_trend_caches([trend], clear_listing=True)` after admin mutations. It clears: SSR HTML (`ssr_trend_*`), JSON detail (`detail_v2_*`), FA API cache (`fa_detail_*`), and FA translation Redis keys (`fa:title:*`, `fa:summary:*`).

### Trend Lookup
`resolve_trend_smart(db, identifier)` handles numeric IDs, slugs (`123-some-slug`), and cluster UUIDs — use it everywhere instead of raw queries.

### Translation Layer (FA)
`app/core/translation.py` has a 3-layer lookup: Redis (24h TTL) → `trends.fa_title`/`trends.fa_summary` DB columns → Gemini API. On Gemini failure the function returns the original text **without caching** so the next request retries. "Stale" Redis entries (where cached value equals the original Turkish) are detected and evicted automatically.

### AI Token Logging
Every Gemini call (summarizer + translation) appends to `ai_monitor_data.csv`. The admin dashboard reads this file. Schema: `timestamp, trend_id, model, input_tokens, output_tokens, duration_sec, category, status, cost_usd`. Use `category` field to distinguish call types (e.g. `FA-Batch`, `FA-Summary`, `FA-Title`).

### FA Translation Invalidation
When `trend.title` or `trend.summary` changes, you must:
1. Set `trend.fa_title = None` and/or `trend.fa_summary = None` on the ORM object before commit
2. Call `clear_fa_cache(trend.id, redis_client)` from `app/core/translation.py` to evict Redis

This is already wired in `summarizer.py` and `routes.py` admin edit endpoints.

## Environment Variables

Key variables in `.env` (service names use Docker network hostnames, not `localhost`):
- `REDIS_HOST=ttw_redis`, `CHROMA_HOST=ttw_chroma`, `POSTGRES_HOST=ttw_postgres`
- `GOOGLE_API_KEY` — Gemini text API (summarizer + translation + merge_worker + x_ai_service)
- `IMAGEN_API_KEY` — Gemini Imagen free tier (image_processor only)
- `OLLAMA_API_URL=http://ttw_ollama:11434/api/generate`
- `BASE_SITE_URL=https://trendiatr.com` — used in canonical URLs and media paths
- TPS thresholds (`THRESHOLD_ADMIN_ALERT=20.0`, `THRESHOLD_AUTO_PUBLISH=35.0`) are also overrideable at runtime via the `system_settings` DB table without restart

## Source Configuration Files

- `app/collectors/rss_sources.txt` — CSV: `name, url, tier, category, speed` (50 feeds)
- `app/collectors/channels.txt` — one Telegram channel username per line (37 channels)

These are hot-reloaded by collectors at runtime; editing them takes effect on the next poll cycle.

## Persian (FA) Feature

Routes at `/fa/`, `/fa/category/<name>`, `/fa/trend/<id>` serve an RTL Persian UI. Cards on `/fa/` fetch titles via `POST /api/fa/translate-titles`. Modals use `GET /api/fa/trends/<id>` which returns `fa_title` and `fa_summary` fields. The full SSR page at `/fa/trend/<id>` calls `translate_title()` and `translate_summary()` server-side. Neither the listing nor SSR FA pages use Redis HTML caching (unlike the Turkish `/trend/<id>` which caches SSR HTML for 600s).

---
> Source: [pdrm55/turkey-trend-watcher](https://github.com/pdrm55/turkey-trend-watcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
