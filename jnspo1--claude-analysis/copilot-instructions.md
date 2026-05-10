## claude-analysis

> Extracts and analyzes tool usage from Claude Code project JSONL logs. Used for permission configuration, security auditing, and understanding Claude Code workflow patterns.

# Task Monitor

Extracts and analyzes tool usage from Claude Code project JSONL logs. Used for permission configuration, security auditing, and understanding Claude Code workflow patterns.

**Storage:** SQLite at `data/cache.db` (4 tables, WAL mode — persistent cache rebuilt from JSONL source files)

## Quick Reference

| Item | Value |
|------|-------|
| Port | 8202 |
| Path prefix | `/activity/` |
| Systemd unit | `task-monitor` |
| Uvicorn module | `app:app` |
| Theme | blue |

## Architecture

**Data flow**: `~/.claude/projects/**/*.jsonl` → `single_pass_parser.py` (single-pass extraction) → `cache_db.py` (SQLite persistent cache) → `app.py` (serve via FastAPI) → `dashboard_template.html` (Chart.js UI)

### Caching Strategy

- **SQLite persistent cache** at `data/cache.db` (WAL mode) stores parsed sessions and pre-computed aggregates
- **Incremental rebuilds**: Only new/changed JSONL files are reparsed (checked by mtime+size)
- **Stale-while-revalidate**: Requests are served instantly from SQLite; background thread rebuilds if cache is >5min old
- **Startup prewarm**: Background rebuild triggers on service start
- **Cold rebuild**: ~8-12s for all files. **Warm rebuild**: <1s (incremental)

### Data Shape

- **Overview tab**: Pre-computed aggregates from `global_aggregates` table (~5KB)
- **Session dropdown**: Lightweight summaries from `session_summaries` table (~30-50KB)
- **Session detail**: Full data from `session_details` table, loaded on demand via `fetch()`

## Key Files

| File | Purpose |
|------|---------|
| `app.py` | FastAPI service with SQLite backend, background rebuilds, tiered data injection |
| `single_pass_parser.py` | Single-pass JSONL parser (merges 5-7 passes into one loop) |
| `cache_db.py` | SQLite schema, CRUD, incremental staleness detection, aggregate computation |
| `session_parser.py` | Original multi-pass parser (still used by CLI scripts) |
| `dashboard_template.html` | HTML/CSS/JS dashboard template (Chart.js, async session loading) |
| `extract_tool_usage.py` | CLI tool extraction to CSV/summary (provides `iter_jsonl()`) |
| `extract_bash_commands.py` | CLI bash command extraction with classification |
| `analyze_permissions.py` | Simulates permission rules against historical tool calls |

## API Endpoints

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/` | Full HTML dashboard (~33KB gzipped) |
| `GET` | `/api/overview` | Pre-computed overview aggregates |
| `GET` | `/api/sessions?project=X` | Lightweight session summaries |
| `GET` | `/api/session/{id}` | Full session detail (lazy loaded) |
| `GET` | `/api/data` | Backward-compatible full payload (deprecated) |
| `GET` | `/api/refresh` | Force incremental cache rebuild |
| `GET` | `/api/rebuild-status` | Check rebuild progress |
| `GET` | `/healthz` | Health check with cache status |

## Configuration

`config.yaml` keys:

| Key | Default | Description |
|-----|---------|-------------|
| `host` | `127.0.0.1` | Bind address |
| `port` | `8202` | Listen port |
| `path_prefix` | `/activity/` | Nginx path prefix |
| `cache_ttl_seconds` | `300` | Cache TTL in seconds (5 minutes) |
| `jsonl_roots` | `["~/.claude/projects"]` | Directories to scan for JSONL files |

## Database

SQLite at `data/cache.db` (WAL mode). Core tables:

| Table | Purpose |
|-------|---------|
| `file_index` | Tracks which JSONL files have been parsed (mtime+size for staleness) |
| `session_summaries` | Lightweight session metadata for dropdown list |
| `session_details` | Full parsed session data (JSON blob, lazy loaded) |
| `global_aggregates` | Pre-computed overview statistics |

## Testing

```bash
./venv/bin/python -m pytest
```

## Development

```bash
# Dev server (auto-reload)
source venv/bin/activate && uvicorn app:app --host 127.0.0.1 --port 8202 --reload

# Restart production
sudo systemctl restart task-monitor

# View logs
sudo journalctl -u task-monitor -f

# CLI scripts (one-off analysis, not used by dashboard)
./venv/bin/python extract_tool_usage.py
./venv/bin/python extract_bash_commands.py
./venv/bin/python analyze_permissions.py
```

### Input

`~/.claude/projects/**/*.jsonl` — Claude Code's project log files.

### CLI Output

- `tool_events.csv` — All tool calls with timestamps, parameters, project context
- `tool_summary.txt` — Aggregated tool usage statistics
- `bash_commands.csv`, `bash_commands_all.txt` — Extracted Bash commands
- `permissions_suggested.yaml` — Suggested permission rules

**Note:** The CLI scripts are for one-off analysis. The web dashboard uses the SQLite cache backend exclusively.

## UI Design System

Theme: **blue** (`<html data-theme="blue">`). Shared CSS: `/shared/pi-design.css`. Skill: `~/.claude/skills/fastapi-ui-design-system.md`.

---
> Source: [jnspo1/claude-analysis](https://github.com/jnspo1/claude-analysis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
