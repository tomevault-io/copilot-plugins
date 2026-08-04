## ha-analyzer

> Flask app that queries InfluxDB for HomeAssistant entity data, runs LLM-powered

# ha-analyzer — Claude Code Guide

Flask app that queries InfluxDB for HomeAssistant entity data, runs LLM-powered
anomaly detection via Ollama, and delivers alerts via Telegram + email.

## Architecture

```
InfluxDB (ha-monitoring, port 8086)
    │  influxdb-client (Flux queries, 30m aggregation by default)
    ▼
ha-analyzer (port 8104, Flask + APScheduler)
    │  Scenarios define: entity list, time window, prompt template, cron schedule
    │  analyzer.py orchestrates: fetch → format → LLM → save → notify
    ▼
Ollama (192.168.0.106:11434, gemma4:31b)
    │  format="json" response, structured anomaly schema
    ▼
Telegram bot + email (philip@davidson.net)
```

Persistent data: SQLite at `/data/db/ha_analyzer.sqlite` (bind-mounted from `./data/db/`).

## Key conventions

- Python 3.12, type hints everywhere, `from __future__ import annotations` on every file
- No `print()` — use `structlog.get_logger(__name__)`
- Ruff for linting, Black for formatting, pytest for tests
- Conventional commits

## CRITICAL: APScheduler + gunicorn

**Gunicorn must run with `--workers 1`.** Multiple workers each start their own
BackgroundScheduler instance, causing every scheduled job to fire N times.
`--worker-class gthread --threads 8` provides concurrency within the single worker.
This is enforced in the Dockerfile CMD — do not change it.

The scheduler is started in `_start_scheduler()` in `app/__init__.py`, with guards
for the Flask dev reloader (`WERKZEUG_RUN_MAIN`) and double-start (`scheduler.running`).

## Security invariants

- All POST routes protected by CSRF (Flask-WTF). The API blueprint (`/api/*`) is
  CSRF-exempt — it's LAN-only and machine-to-machine.
- No user authentication (LAN-only deployment). Add if ever exposed externally.
- MAIL_PASSWORD and TELEGRAM_BOT_TOKEN must never be committed.

## Services

| File | Purpose |
|------|---------|
| `influxdb_service.py` | Flux queries; `get_entity_data()` + `list_entity_ids()` |
| `ollama_service.py` | httpx POST to `/v1/chat/completions`; JSON format; parse response |
| `analyzer.py` | Orchestration: fetch → format → LLM → save → notify; `DEFAULT_PROMPT_TEMPLATE` |
| `notifier.py` | Telegram httpx + smtplib STARTTLS email |
| `scheduler.py` | APScheduler job registration/removal; `start_scheduler()`, `add_scenario_job()` |

## Scenario entity IDs

Entity IDs must match InfluxDB tag values exactly (same as HA entity IDs, e.g.
`sensor.living_room_temperature`). Use InfluxDB Explore or `/api/scenarios` to
browse available IDs. Flux measurement name = HA domain (e.g. `sensor`, `climate`);
always filter by `entity_id` tag, not measurement.

## Build / run

```bash
# Local dev (install deps first)
uv pip install -e ".[dev]"
npm install
npm run watch:css
flask --app "app:create_app('development')" run --port 8104 --debug

# Lint / format
ruff check app && black --check app

# Tests
pytest -x -q

# Docker
docker compose -f docker/docker-compose.yml up --build
```

## After any model change

```bash
flask db migrate -m "describe change"
flask db upgrade
# Commit the new migration file
```

## Deployment

Deploy to Services VM at `/opt/docker/ha-analyzer/`.
ha-monitoring must be deployed first — ha-analyzer joins its Docker network.

```bash
rsync -a --exclude='.git' . philip@192.168.0.107:/opt/docker/ha-analyzer/
ssh philip@192.168.0.107 "cd /opt/docker/ha-analyzer && docker compose -f docker/docker-compose.yml build && docker compose -f docker/docker-compose.yml up -d"
ssh philip@192.168.0.107 "docker exec ha-analyzer flask db upgrade"
```

## Git

- Remote: `https://github.com/pmd1965/ha-analyzer`
- Branch: `main`
- Commit style: conventional commits (`feat:`, `fix:`, `chore:`, `docs:`, `test:`, `refactor:`)
- Push after every meaningful working state
- Never commit `.env` — document all vars in `.env.example`

---
> Source: [pmd1965/ha-analyzer](https://github.com/pmd1965/ha-analyzer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
