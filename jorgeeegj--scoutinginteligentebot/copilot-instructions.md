## scoutinginteligentebot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

ScoutingInteligenteBot is a Telegram bot for football scouting, built on a multi-agent LangGraph pipeline with RAG. It answers natural-language queries about ~12,000 players using FAISS vector search + hard filters + scoring + LLM explanation + chart generation.

## Commands

All execution is via Docker. The image must be built first:

```bash
docker compose build --no-cache --pull
```

### Run the bot
```bash
docker compose up -d bot
docker compose logs -f bot
```

### ETL pipeline
```bash
# Full ETL (collect FotMob + Transfermarkt + merge + index)
docker compose run --rm etl python -m scouting.etl.main_etl

# Skip steps selectively
docker compose run --rm etl python -m scouting.etl.main_etl --skip-fotmob --skip-tm --skip-merge

# Individual steps
docker compose run --rm etl python -m scouting.etl.collect_fotmob
docker compose run --rm etl python -m scouting.etl.collect_transfermarkt
docker compose run --rm etl python -m scouting.etl.merge
docker compose run --rm etl python -m scouting.etl.vector_store_indexing
```

### FotMob token (required before each FotMob scrape)
The `X_MAS_TOKEN` in `.env` must be refreshed manually before running `collect_fotmob`:
> Open FotMob in browser → any player page → DevTools → Network → Fetch/XHR → click any element → find `currency` request → look for `X-Mas` header → copy value → paste into `.env` as `X_MAS_TOKEN` (no quotes).

## Architecture

The repo root (`proyectosjg/`) is the Python project; `src/scouting/` is the package.

### Multi-agent pipeline (`src/scouting/agents/`)

LangGraph orchestrates agents 0–4 in sequence. State flows through `PipelineState`.

| Agent | File | Role |
|---|---|---|
| Supervisor | `pipeline.py` | Validates query, detects language, normalizes to Spanish |
| Agent 0 | `agent0.py` | FAISS vector retrieval — returns top-3500 candidates as Polars DataFrame |
| Agent 1 | `agent1.py` | Hard filters: age, market value, position, foot, height, contract, nationality, league |
| Agent 2 | `agent2.py` | Selects 8 key metrics (embeddings + intent rules), calculates weighted score with minutes adjustment and league coefficient |
| Agent 3 | `agent3.py` | LLM explanation (OpenAI) in the user's language |
| Agent 4 | `agent4.py` | Generates Plotly radar collage + mplsoccer pizza charts |
| Bot | `telegram_bot.py` | Polling loop; offset persisted in `/data/last_update_id.txt` |

**Embedding model**: `intfloat/multilingual-e5-base` (SentenceTransformers)
**LLM**: OpenAI (`gpt-4o` for supervisor, `gpt-4o-mini` for agent 3 by default)
**Vector store**: FAISS (separate indices for `jugadores` and `porteros`)

### ETL pipeline (`src/scouting/etl/`)

1. `collect_fotmob.py` — scrapes FotMob API for player statistics
2. `collect_transfermarkt.py` — scrapes Transfermarkt for contract/market value data
3. `merge_data.py` — joins both sources, splits into `db_jugadores.json` / `db_porteros.json`
4. `vector_store_indexing.py` — builds FAISS indices + metadata JSON files

### Data layout (not versioned)
```
data/processed/
  merged/
    db_jugadores.json
    db_porteros.json
  indices/current/
    faiss_jugadores.index
    faiss_porteros.index
    metadata_jugadores.json
    metadata_porteros.json
```

## Environment Variables

Copy `.env.example` to `.env` and fill in:

| Variable | Default | Description |
|---|---|---|
| `OPENAI_API_KEY` | — | Required |
| `TELEGRAM_BOT_TOKEN` | — | Required |
| `OPENAI_MODEL_SUPERVISOR` | `gpt-4o` | Supervisor/pipeline LLM |
| `INDICES_DIR` | `/data/processed/indices/current` | Path to FAISS indices |
| `KAL_CHROME_PATH` | `/usr/bin/google-chrome-stable` | Chrome for Kaleido PNG export |
| `X_MAS_TOKEN` | — | FotMob API token (must refresh manually) |

## Key Implementation Notes

- **Polars** (not pandas) is used throughout for DataFrames.
- `common.py` provides `jlog()` for structured JSON logging and `env_str/env_int` helpers — use these for all config access.
- Agent 2 treats metrics where lower is better (e.g., `Dispossessed`, `Fouls committed`, `Dribbled past`) differently — they skip raw value in scoring.
- Agent 2 applies a **league coefficient** (Big-5 ≈ 1.0, lower leagues < 1.0) to normalize scores.
- Agent 4 depends on `mplsoccer` and `Pillow`; if unavailable, pizza charts are skipped and only Plotly radars are generated.
- PNG export via Kaleido requires Chrome (`KAL_CHROME_PATH`); if missing, rebuild the Docker image.
- `src/` is mounted as a volume in docker-compose, so local edits are reflected without rebuilding.

---
> Source: [jorgeeegj/ScoutingInteligenteBot](https://github.com/jorgeeegj/ScoutingInteligenteBot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->
