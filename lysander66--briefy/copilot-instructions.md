## briefy

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Briefy** is an AI-powered daily news briefing service that aggregates trending topics from Chinese news platforms, generates AI summaries, and produces text-to-speech audio briefings. It runs as a FastAPI web service with a 30-minute scheduled fetch cycle.

## Commands

```bash
# Install dependencies (requires uv)
uv sync

# Run the service
uv run main.py
# Service available at http://127.0.0.1:8000

# Run individual test scripts (for manual testing of components)
uv run test-fetch.py        # Test fetchers
uv run test-aggregator.py   # Test aggregation
uv run test-selector.py     # Test news selection
uv run test-generator.py    # Test summary generation
uv run test-tts.py          # Test text-to-speech
```

## Configuration

Copy `.env.example` to `.env`. Key variables:

- `ENABLE_SUMMARY=1` — enable AI summary + TTS (default: 0, fetch-only mode)
- `READER_API_KEY` — API key for fetching article content
- `LLM_API_KEY`, `LLM_MODEL`, `LLM_API_BASE` — LLM config via LiteLLM (supports OpenAI, DeepSeek, GLM, etc.)

## Architecture

### Data Flow

```
Scheduler (every 30min)
  → fetch_all_sources()       # Fetches from all registered sources
    → CacheStorage.save()     # Saves raw data to temp/<source_id>/YYYYMMDD_HHMM.json
  → DailyAggregator.generate() # Reads temp/, aggregates, writes data/YYYY-MM-DD.md
  → generate_daily_summary()   # (if ENABLE_SUMMARY=1)
    → select_top_news()       # Picks top N items from the day's aggregated data
    → fetch_contents_batch()  # Fetches article markdown via Reader API
    → generate_summaries()    # LLM summarization via LiteLLM
    → generate_audio()        # TTS via edge-tts → data/audio/YYYY-MM-DD.mp3
    → saves data/summaries/YYYY-MM-DD.json
```

### Key Modules

- **`fetcher/`** — News source adapters. Each fetcher extends `BaseFetcher` (abstract) with `source_id` and `async fetch() -> List[Trend]`. All fetchers self-register in `fetcher/__init__.py` via `FetcherRegistry.register()`. To add a new source, create a new fetcher file and add it to `__init__.py`.

- **`storage/`**
  - `cache.py` — `CacheStorage`: saves raw fetch results to `temp/<source_id>/YYYYMMDD_HHMM.json`
  - `aggregator.py` — `DailyAggregator`: reads all cache files for a date, deduplicates/scores topics, writes `data/YYYY-MM-DD.md`. Scoring uses appearance count (60%) + rank weight (40%), or raw scores if available.

- **`summary/`**
  - `selector.py` — selects top N items from the day's markdown file
  - `reader.py` — fetches full article content via Reader API
  - `client.py` — LLM summarization via LiteLLM
  - `tts.py` — text-to-speech via edge-tts
  - `generator.py` — orchestrates the full summary pipeline

- **`config/config.py`** — Single `Config` dataclass loaded from env via `cfg` global. Paths default to `data/` and `temp/` relative to the working directory.

- **`web/`** — `render.py` generates the HTML dashboard from aggregated markdown and summary JSON. Templates in `web/templates/`.

### File Storage Layout

```
temp/<source_id>/YYYYMMDD_HHMM.json   # Raw fetched data (cache)
data/YYYY-MM-DD.md                     # Aggregated daily markdown
data/summaries/YYYY-MM-DD.json         # AI summaries + metadata
data/audio/YYYY-MM-DD.mp3             # TTS audio
```

### API Endpoints

- `GET /` — HTML dashboard (optional `?date=YYYY-MM-DD`)
- `GET /api/summary/{date}` — JSON summary data
- `GET /api/audio/{date}` — MP3 audio file

## Deployment

SSH host alias: `briefy-server`. App directory on server: `/opt/briefy`. Managed via systemd unit `briefy.service`.

```bash
# Deploy (sync code, uv sync, restart service)
./scripts/deploy.sh              # default host: briefy-server
./scripts/deploy.sh myhost       # specify another SSH host

# View logs
ssh briefy-server 'sudo journalctl -u briefy.service -n 50'   # last 50 lines
ssh briefy-server 'sudo journalctl -u briefy.service -f'       # follow live

# Service status
ssh briefy-server 'sudo systemctl status briefy'
```

First-time server setup: see `deploy/README.md`.

---
> Source: [Lysander66/briefy](https://github.com/Lysander66/briefy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
