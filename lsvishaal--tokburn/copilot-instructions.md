## tokburn

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
make test          # Run full test suite
make lint          # Check style (ruff check + format --check)
make fix           # Auto-fix lint/format issues
make dev           # Start server in foreground (verbose, Ctrl+C to stop)
make serve         # Start server in background on port 8391
make stop          # Stop the background server
make scan          # CLI scan of sessions without starting the server

# Run a single test
uv run pytest tests/test_core.py::test_calculate_cost_known_model -v

# Install deps
uv sync --dev
```

## Architecture

TokBurn is a local analytics dashboard that reads AI coding agent session logs, calculates costs, and surfaces waste-detection insights. It never phones home — all data is read from disk at `~/.claude/projects/`.

### Data flow

```
~/.claude/projects/<encoded-path>/<session-id>.jsonl
        ↓
ClaudeCodeParser   (parsers/claude_code.py)
  discover_sessions() → list of JSONL paths
  parse_session()     → Session dataclass
        ↓
Session / Message / ToolCall  (parsers/base.py)
  — token counts, tool calls, timestamps
        ↓
calculate_cost()  (cost.py)
  — applies MODEL_PRICING / CACHE_PRICING tables
        ↓
analyze_sessions()  (analyzer.py)
  — emits Insight objects (repeated_reads, floundering, cost_outlier, long_session)
        ↓
FastAPI server  (server.py)
  — 30s in-memory cache; REST endpoints feed the dashboard
```

### Key design details

**Parser (`parsers/claude_code.py`)** — Claude Code embeds token usage in two layouts depending on version: `record.usage` (newer) and `record.message.usage` (older). The parser checks both. Session files live under `~/.claude/projects/<encoded-cwd>/` in three possible layouts: `<session>.jsonl`, `sessions/<session>.jsonl`, and `<session>/subagents/<agent>.jsonl`. Project paths are encoded by replacing `/` with `-`; `_decode_project_path()` reverses this.

**Cost engine (`cost.py`)** — Prices are per-million tokens in `MODEL_PRICING` / `CACHE_PRICING`. Model IDs from session files carry date suffixes (e.g., `claude-sonnet-4-5-20250929`); `normalize_model_id()` strips the 8-digit suffix before lookup, then falls back to prefix matching for variant model IDs.

**Analyzer (`analyzer.py`)** — Four independent waste detectors, each returning `list[Insight]`. `detect_cost_outliers` is cross-session and requires ≥5 sessions. Insights are sorted high→low severity then by estimated waste tokens.

**Server (`server.py`)** — Global `_sessions_cache` is refreshed every 30 seconds. The static dashboard (`src/tokburn/static/`) is served at `/`; API is at `/api/*`. FastAPI auto-docs are at `/docs`.

### Adding a new parser

1. Subclass `BaseParser` (parsers/base.py) and implement `discover_sessions()` and `parse_session()`.
2. Set `agent_name` to a unique string (e.g., `"aider"`).
3. Wire it into `server.py`'s `_get_sessions()` alongside `ClaudeCodeParser`.

### Static assets

- `src/tokburn/static/favicon.svg` — flame emoji SVG favicon, served at `/static/favicon.svg`
- `docs/` — screenshots for README (`dashboard.png` hero, `insights.png` detail); referenced in README via absolute GitHub raw URLs so they render on both GitHub and PyPI
- README style: no em-dashes, minimal AI-looking structure

### Ruff config

Line length 100, Python 3.11 target. Rules: `E W F I N UP B SIM TCH`. `E501`, `TC001`, `TC003`, `SIM102` are ignored. Imports must be isort-sorted with `tokburn` as first-party.

---
> Source: [lsvishaal/tokburn](https://github.com/lsvishaal/tokburn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
