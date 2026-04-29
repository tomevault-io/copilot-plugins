## hermes-hud

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Hermes HUD is a self-improvement dashboard for the Hermes Agent. It reads data from `~/.hermes/` to display learning progress, memory state, skill growth, session history, and growth deltas. It runs as an interactive TUI built with Textual.

## Running

```bash
make dev                   # Create venv + install editable with neofetch extras
source venv/bin/activate
hermes-hud                 # Launch the TUI
hermes-hud --help          # See all options
```

Or manually: `python3 -m venv venv && source venv/bin/activate && pip install -e .`

Requires Python 3.11+. Override Python version: `make venv PYTHON=python3.12`

### CLI modes

```bash
hermes-hud --text          # Print HUDState summary to stdout (useful for scripting)
hermes-hud --snapshot      # Save snapshot to ~/.hermes/.hud/snapshots.jsonl
hermes-hud --ai            # AI awakening neofetch
hermes-hud --br            # Blade Runner neofetch
hermes-hud --fsociety      # fsociety/Mr. Robot neofetch
hermes-hud --anime         # Mewtwo ASCII art neofetch
```

## Testing

```bash
make test                                          # Run full test suite
python -m pytest tests/test_integration.py -v     # Single file
python -m pytest -k "memory" -v                   # Filter by name
```

`tests/conftest.py` provides `fake_hermes_home` and `fake_projects_dir` fixtures that build a realistic `~/.hermes/` tree (sqlite3 state.db, config.yaml, YAML files, git repos). Tests use `monkeypatch` for env vars and module attributes.

## Architecture

```
hermes_hud/
├── hud.py              # Main Textual App + CLI entry point
├── collect.py          # Orchestrates collectors into HUDState
├── models.py           # Dataclasses (HUDState, MemoryState, etc.)
├── snapshot.py         # Snapshot/diff tracker
├── neofetch_base.py    # Shared neofetch data collection + display utils
├── neofetch_{ai,anime,br,fsociety}.py
├── collectors/         # One collector per data source
└── widgets/            # One Textual panel per tab
```

**Data flow:** `collectors/ → collect.py → models.py → widgets/ → hud.py`

## Key Patterns

- **Proper package**: All imports are relative (e.g., `from .models import ...`, `from ..collectors.cron import ...`). No sys.path hacks.
- **Entry point**: `hermes_hud.hud:main` registered in pyproject.toml `[project.scripts]`
- **Environment variables**: `HERMES_HOME` (agent data dir), `HERMES_HUD_PROJECTS_DIR` (projects scan dir), `HERMES_HUD_NOBOOT` (skip animation)
- **Centralized paths**: All path resolution goes through `collectors/utils.py` (`default_hermes_dir()`, `default_projects_dir()`)
- 4 registered themes (`hermes-ai`, `hermes-blade-runner`, `hermes-fsociety`, `hermes-anime`) in `hud.py`
- Tabs are lazy-loaded on first switch via `on_tabbed_content_tab_activated`
- **Concurrent collection**: `collect.py` uses `ThreadPoolExecutor(max_workers=4)` to parallelize collectors; config collector must complete first (memory limits depend on it). `hud.py._load_data()` uses `max_workers=8` for dashboard refresh.
- **Graceful degradation**: `_check_hermes_data()` warns but doesn't exit if `~/.hermes/` is missing — all panels render empty, useful for demo/dev.
- **Snapshot system**: `snapshot.py` captures point-in-time metrics for diff analysis. Snapshots stored as JSONL at `~/.hermes/.hud/snapshots.jsonl`.

---
> Source: [joeynyc/hermes-hud](https://github.com/joeynyc/hermes-hud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
