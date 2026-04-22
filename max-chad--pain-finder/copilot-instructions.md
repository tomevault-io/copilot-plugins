## pain-finder

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run
python main.py

# Quality gates (must all pass before merging)
ruff check .
mypy .
pytest --cov=. --cov-fail-under=80 -q

# Run a single test file
pytest tests/test_pipeline.py -q

# Run a single test by name
pytest tests/test_pipeline.py::test_name -q

# Docker
docker compose up -d --build
```

## Architecture

`main.py` is the composition root. It constructs all services and wires them together — nothing is imported and auto-discovered. The dependency graph flows:

```
config → all services
db (Database) → pipeline, classifier, budget_guard, clusterer, gtm_generator, export_service, scheduler
openrouter (OpenRouterClient) → classifier, clusterer, gtm_generator
budget_guard (BudgetGuard) → openrouter, pipeline, scheduler
pipeline (AnalysisPipeline) → bot (via analyze_fn/deep_dive_fn/digest_fn callbacks)
scheduler (MonitoringScheduler) → bot (via reload_jobs_fn)
bot (PainFinderBot) → Telegram app lifecycle
```

**Data flow for a Reddit analysis:**
1. `scheduler.py` fires a monitored-subreddit job → calls `analyze_and_notify()` in `main.py`
2. `pipeline.analyze_subreddit()` → `scraper.fetch_posts()` → `classifier.classify_batch()`
3. Each post with `willingness_to_pay >= DEEP_DIVE_WTP_THRESHOLD` auto-triggers `pipeline.run_deep_dive()`
4. Results persisted to SQLite via `db.py`; report JSON written to `REPORTS_DIR`
5. Telegram notification sent via `bot.app.bot.send_message()`

**LLM budget guard:** Every LLM call path checks `budget_guard.ensure_can_spend()` first. If the daily cap is hit, `runtime_flags.llm_paused` is set in the DB, the scheduler reloads to skip LLM jobs, and a Telegram alert fires. `/resume` sets a temporary override until the next UTC midnight.

**Classifier modes** (`CLASSIFIER_MODE`): `legacy` = original keyword+LLM approach; `b2b` = strict B2B-only scoring; `dual` = runs both and merges (recommended, provides fallback when strict schema parse fails).

**External sources:** Reddit (PRAW with JSON fallback), Hacker News (Algolia API via `scraper_hn.py`), and review sites (`scraper_reviews.py` with BeautifulSoup). All sources feed into `pipeline.analyze_external_posts()` with a `source` tag that propagates to `pain_points.source`.

**Macro clustering** (`clusterer.py`): Uses local cosine-similarity embedding clustering on historical high-signal items, then calls the LLM to generate trend labels. Runs on a weekly UTC schedule or on-demand via `/macro`.

## Key Conventions

- All DB access is async via `aiosqlite`. Schema is managed with additive migrations in `db.py` — never drop or rename existing columns.
- `openrouter.py` enforces strict JSON schema responses from the LLM. Parse failures in `dual` mode fall back gracefully; in `b2b`/`legacy` mode they raise.
- `config.py` reads all env vars at import time. JSON env vars (`HN_KEYWORDS_JSON`, `REVIEW_TARGETS_JSON`, `OPENROUTER_MODEL_PRICING_JSON`) are parsed via `parse_json_env()` with a safe default fallback.
- Reports are written atomically: write to `.tmp` then `os.replace()`.
- Telegram inline callback IDs follow the format `action:post_id:scope` — the `post_id` itself can contain `:`, so parsers must split on the first two `:` only.
- `pytest.ini` sets `asyncio_mode = auto`; all async tests just use `async def` without explicit event loop setup.

## Debug Playbooks

See `AGENTS.md` for detailed runbooks on: Reddit 429s, OpenRouter failures, budget cap incidents, Google Sheets export failures, and SQLite contention.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/max-chad) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
