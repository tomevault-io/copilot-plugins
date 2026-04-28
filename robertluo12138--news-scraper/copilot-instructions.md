## news-scraper

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

A Python 3.11+ MVP that generates a daily Chinese Markdown news digest covering the food delivery / restaurant / instant retail / local life industry (China-focused, some global references). It fetches RSS metadata, scores relevance with keyword lists, stores everything in SQLite, and writes a report to `reports/YYYY-MM-DD.md`.

## Run / develop

```bash
pip install -r requirements.txt
cp .env.example .env
python main.py --date today                 # full fetch + report
python main.py --date 2026-04-14             # specific date
python main.py --date today --skip-fetch     # regenerate report only
```

There is no test suite, linter config, or CI yet. When adding one, keep the MVP runnable with a single `python main.py` invocation.

## Architecture (read this before editing)

The pipeline is linear and lives in `src/pipeline.py::run_pipeline`:

1. `config.load_sources` reads `config/sources.yaml` (only `enabled: true` sources).
2. `fetcher.fetch_source` pulls each RSS feed via `feedparser` and returns **metadata only** — title, link, short cleaned summary, published date. **Do not extend this to fetch full article bodies** — that's an explicit copyright constraint of the project.
3. `pipeline._ingest` runs the dedupe + scoring + insert loop:
   - URL dedupe via `db.url_exists` (backed by `UNIQUE` on `articles.url`) + `dedupe.normalize_url` (strips fragment).
   - Near-duplicate title dedupe via `dedupe.is_near_duplicate_title` (rapidfuzz token-set ratio, threshold 85) against **titles already seen for the same target date**.
   - `scorer.score_article` computes a total score, best category, and matched keywords.
4. `db.fetch_articles_for_date` selects rows where `substr(COALESCE(published_at, fetched_at), 1, 10) == target_date` and `score >= MIN_SCORE`.
5. `reporter.generate_report` writes the Chinese Markdown, grouped by category in a fixed order.

### Key design choices worth knowing

- **Metadata only, by policy.** `fetcher.SUMMARY_MAX_CHARS = 400` caps summary length. Never store article bodies.
- **Date bucket = `published_at` falling back to `fetched_at`.** Feeds without a publish date are still reportable the day they're fetched.
- **Score = sum across categories; category = single best category.** See `src/scorer.py::CATEGORIES`. Tune keywords there — each category has a weight multiplier.
- **`MIN_SCORE` filters the report, not the DB.** Everything fetched is persisted; filtering happens at report generation so you can re-run with `--skip-fetch` after adjusting `MIN_SCORE` in `.env`.
- **One bad feed shouldn't kill the run.** `pipeline._fetch_all` catches per-source exceptions and logs them.

## Common edit patterns

- **Adding / removing sources:** edit `config/sources.yaml`. No code changes needed.
- **Tuning relevance:** edit `CATEGORIES` in `src/scorer.py` (add terms or adjust weights). Keep Chinese and English terms in the same list — the matcher lowercases and does substring checks, which is fine for Chinese.
- **Changing the report format:** edit `src/reporter.py`. Category labels and order live in `CATEGORY_LABELS` / `CATEGORY_ORDER`.
- **Adding an HTML source type:** `fetcher.fetch_source` currently warns and skips `type: html` sources. Implement a new branch there; keep to metadata only (title + link + optional short teaser).

## Things to keep in mind

- Respect each source's ToS and robots.txt. The project's README makes the copyright stance explicit — don't quietly undo it.
- The CLI signature (`python main.py --date today`) is part of the spec — don't rename the flag without updating README and CLAUDE.md.
- SQLite path and reports path are configurable via `.env` (`DB_PATH`, `REPORTS_DIR`); don't hardcode them.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Robertluo12138) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-15 -->
