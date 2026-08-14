## lol-addicted

> - `src/collector.py` calls Riot APIs and writes immutable JSON to `data/raw/`.

# Repository Guidelines

## Project Structure & Module Organization

- `src/collector.py` calls Riot APIs and writes immutable JSON to `data/raw/`.
- `src/processor.py` parses raw match and timeline files into `data/lol.duckdb`.
- `src/features.py` computes analytics; `src/models.py` trains K-Means and persists labels.
- `tests/` mirrors source modules; fixtures live in `tests/fixtures/`.
- `notebooks/` contains exploration. Never import notebooks from `src/`.
- `scripts/workflow.ps1` is the developer entry point.

Raw and processed data retain every role. Analytics and models currently use Season 16 `MIDDLE` games only. All-role support requires role-aware opponent extraction and a DuckDB rebuild.

The canonical `FEATURE_COLS` list lives in `src/models.py`; never derive it from DataFrame columns. Keep API access in `collector.py`, parsing and database writes in `processor.py`, and feature logic in `features.py`.

## Build, Test, and Development Commands

Run from the repository root in PowerShell:

```powershell
.\scripts\workflow.ps1 sync      # Create .venv and install dependencies
.\scripts\workflow.ps1 collect   # Download configured Riot data
.\scripts\workflow.ps1 process   # Load new raw files into DuckDB
.\scripts\workflow.ps1 features  # Rebuild the mid-only feature matrix
.\scripts\workflow.ps1 models    # Retrain clusters and persist labels
.\scripts\workflow.ps1 refresh   # Run the complete incremental pipeline
.\scripts\workflow.ps1 rebuild   # Recreate derived data from raw files
.\scripts\workflow.ps1 test      # Run pytest
.\scripts\workflow.ps1 smoke     # Five-match live pipeline check; needs a valid Riot key
.\scripts\workflow.ps1 deploy-db # Create the Streamlit database snapshot
.\scripts\workflow.ps1 dashboard # Start the local Streamlit app
python -m ruff check src tests dashboard scripts   # Lint Python
```

Commands stop on the first failed native process. Use `rebuild` after parser or schema changes.

## Coding Style & Testing

Use four-space indentation, function type hints, focused public docstrings, `snake_case` names, and `UPPER_CASE` constants. Prefer `pathlib.Path`, parameterized DuckDB queries, and specific exceptions.

Tests use `pytest` and must be deterministic and fixture-backed. Name files `test_<module>.py` and tests `test_<behavior>`. Directly test dashboard-facing feature behavior, including season and role boundaries. Never call Riot APIs in unit tests.

## Commit & Pull Request Guidelines

Use focused commits with short imperative subjects such as `fix: scope matchups to mid`. Pull requests should describe behavior changes, verification commands, and schema or generated-data effects. Link issues and include screenshots only for visual changes.

## Security & Data Rules

Never commit `.env`, Riot keys, `data/raw/`, or local `*.duckdb`; `data/lol_deploy.duckdb` is the only exception. Review that deployment database before publishing because timestamps plus champion/version data can still be identifying even after match IDs are replaced with surrogates. The dashboard must open it read-only.

Preserve Riot routing: Account-v1 uses `asia.api.riotgames.com`, and Match-v5 uses `sea.api.riotgames.com`.

---
> Source: [vinhnqgcs220115/LoL-addicted](https://github.com/vinhnqgcs220115/LoL-addicted) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
