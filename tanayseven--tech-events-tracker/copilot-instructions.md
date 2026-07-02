## tech-events-tracker

> - `main.py` — entrypoint: runs the Scrapy spider via `CrawlerProcess`

# Claude Code guidance

## Project layout

- `main.py` — entrypoint: runs the Scrapy spider via `CrawlerProcess`
- `tech_events/` — Scrapy project
  - `spiders/events.py` — fetches every URL in `events_list.yaml`
  - `pipelines.py` — `ClaudeExtractionPipeline` (HTML → structured JSON via Claude) + `YamlWriterPipeline` (upserts into `event/<year>.yaml`)
  - `items.py` — `PageItem` (event_name, url, html)
  - `settings.py` — Scrapy settings (delays, headers, pipeline order)
- `app.py` — Flask + Frozen-Flask app; groups events by `series` and passes to template
- `templates/index.html` — Jinja2 template with search/filter UI; SVG icons inlined via macros
- `event/<year>.yaml` — event data (both managed and manually added entries)
- `events_list.yaml` — source of truth for which events to track (url + name)
- `extractors.yaml` — auto-generated; CSS selectors per series for local extraction (skip Claude on re-runs)
- `pyproject.toml` / `uv.lock` — Python dependencies managed by uv
- `mise.toml` — installs uv via mise

## Key design decisions

**`events_list.yaml` is the source of truth for scope.** Only events listed there are ever written or updated. Manually added entries (entries whose `series` doesn't match any managed event) are preserved across re-runs via `_upsert()` in `pipelines.py`. Currently tracks ~100 events across India-specific and international categories (cloud/DevOps, security, AI/ML, language communities, hardware, networking, etc.).

**Two-stage pipeline:** Scrapy handles HTTP (retries, delays, robots.txt), `ClaudeExtractionPipeline` handles extraction. On each item it first tries stored CSS selectors from `extractors.yaml` via `parsel`; only calls Claude when selectors are missing or return no date. Claude returns both event data and updated selectors in one call (`{"events": [...], "selectors": {...}}`). `YamlWriterPipeline` upserts on `close_spider`.

**`extractors.yaml`:** auto-generated file storing per-series CSS selectors (parsel syntax). Has `container` (repeating-block selector or null) and `fields` mapping each `TechEvent` field to a CSS selector or null (for interpretation fields like `mode`, `scope`). Claude regenerates selectors whenever local extraction fails. Console labels `[local]` / `[claude]` show which path was taken.

**`from_crawler()` pattern:** Both pipelines use `from_crawler()` to store the crawler and access `self.crawler.spider` — the `spider` argument on pipeline methods is deprecated in the current Scrapy version.

**Upsert dedup keys:** `(series.lower(), date)` pair AND `event_website` URL. An entry is skipped if either key has been seen, preventing duplicates when multiple events share a URL.

**`series` field:** stamped in `ClaudeExtractionPipeline` via `e["series"] = name` using the `events_list.yaml` name, keeping dedup stable regardless of how Claude phrases the event name.

**`{year}` URL templates:** URLs in `events_list.yaml` may contain `{year}` anywhere — path segment, subdomain, query param. The spider replaces it with the current year at crawl time (e.g. `https://{year}.pyconfhyd.org/` → `https://2026.pyconfhyd.org/`). Use this for events whose URL changes each year.

**Upsert across runs:** existing managed entries whose fresh scrape is skipped (e.g. 404, same-URL dedup) are preserved in `_upsert()` rather than dropped. Only entries explicitly returned by Claude as new/changed are updated.

**Date handling:** entries with missing or unparseable dates are treated as current-year (future) events by `_is_current_year()`.

**Model:** `claude-sonnet-4-6` at $3.00/$15.00 per 1M tokens (input/output).

## Running things

```bash
export ANTHROPIC_API_KEY=sk-ant-...
uv run python main.py            # fetch events → event/<year>.yaml
uv run scrapy crawl events       # equivalent, via Scrapy CLI
uv run python app.py             # dev server on :5000
uv run python app.py freeze      # build static site → build/
```

## Git commits

Never add a `Co-Authored-By: Claude` trailer to commit messages.

## Colour palette

- Danube `#5992c6`
- Cocoa Brown `#31241f`
- Shilo `#e9b8c9`
- Torea Bay `#0a2a92`

---
> Source: [tanayseven/tech-events-tracker](https://github.com/tanayseven/tech-events-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-02 -->
