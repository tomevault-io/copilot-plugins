## ideaforge

> cd ~/projects/ideaforge

# IdeaForge - Agents Operational Guide

## Build & Install
```bash
cd ~/projects/ideaforge
python -m venv .venv
source .venv/bin/activate
pip install -e ".[dev]"
```

## Run
```bash
# Initialize database
ideaforge init

# Full pipeline
ideaforge run

# Individual steps
ideaforge harvest
ideaforge synthesize
ideaforge score
ideaforge status
ideaforge export <id>
```

## Test
```bash
pytest tests/ -v
```

## Lint & Format
```bash
ruff check src/ tests/
ruff format src/ tests/
mypy src/
```

## Environment
Requires `ANTHROPIC_API_KEY` in `.env` or environment for synthesize/score commands.
Reddit harvesting needs no credentials.

## File Layout
- `src/ideaforge/` - Main package
- `tests/` - pytest test suite
- `data/` - SQLite database (created at runtime)

---
> Source: [m2ai-st-metro/ideaforge](https://github.com/m2ai-st-metro/ideaforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
