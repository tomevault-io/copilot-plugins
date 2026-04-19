## against-wind

> running commands, Development Commands,Tests, Docker rules


# Against Wind Project Rules

## Import Structure
- Always use absolute imports from project root: `api.app.*` instead of `app.*`
- Examples:
  - `from api.app.core.config import get_settings`
  - `from api.app.main import app`
  - `from api.app.services.analyze import AnalysisService`

## Package Manager
- Use `uv` for all Python operations, never `pip`
- Commands: `uv add {package}`, `uv sync`, `uv run python xxx`
- Dependencies managed in [pyproject.toml](against-wind/pyproject.toml)

## Development Commands (run from root directory)
- API server: `uv run uvicorn api.app.main:app --reload`
- Tests: `uv run pytest api/tests/`
- Worker: `uv run python -m api.app.worker.worker`
- Dependencies: `uv sync`

## Docker Configuration
- Separate dockerfiles: [api.dockerfile](against-wind/api.dockerfile) and [ui.dockerfile](against-wind/ui.dockerfile)
- Both build from root context
- API dockerfile uses `api.app.main:app` as entry point

## Test Configuration
- pytest config in [pyproject.toml](against-wind/pyproject.toml) with:
  - `pythonpath = ["."]`
  - `testpaths = ["api/tests"]`
- Never use `sys.path.insert()` in test files

## Project Structure
- Mono-repo with `/api` and `/ui` directories
- All Python code uses absolute imports from root
- Docker files at root level, not in subdirectories

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AnsonDev42) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
