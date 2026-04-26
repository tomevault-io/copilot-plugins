## pi-couch-hound

> Pi Couch Hound — Raspberry Pi-powered dog detector that monitors a couch and triggers configurable actions when a dog is detected. Python backend (FastAPI) + React TypeScript frontend (Vite).

# CLAUDE.md

## Project Overview

Pi Couch Hound — Raspberry Pi-powered dog detector that monitors a couch and triggers configurable actions when a dog is detected. Python backend (FastAPI) + React TypeScript frontend (Vite).

## Repository Structure

- `couch_hound/` — Python backend package
  - `api/` — FastAPI app factory, route modules, Pydantic schemas
  - `actions/` — Action executor base class and implementations
  - `config.py` — YAML config loader/saver with Pydantic validation
  - `main.py` — Entry point (uvicorn runner)
- `frontend/` — React TypeScript SPA (Vite, Tailwind, TanStack Query, Zustand)
- `tests/` — pytest test suite
- `SPEC.md` — Full technical specification (source of truth for API contracts, config schema, architecture)

## Development Commands

### Backend

This project requires Python 3.12 and uses [uv](https://docs.astral.sh/uv/) for dependency management:

```bash
# Install uv (if not already installed)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install all dependencies (creates .venv automatically)
uv sync --extra dev

# Lint
uv run ruff check couch_hound/ tests/

# Format check
uv run ruff format --check couch_hound/ tests/

# Auto-format
uv run ruff format couch_hound/ tests/

# Type check
uv run mypy couch_hound/

# Run tests
uv run pytest tests/ -v

# Run tests with coverage
uv run pytest tests/ -v --cov=couch_hound --cov-report=term-missing

# Run dev server
uv run uvicorn couch_hound.api.app:create_app --factory --port 8080 --reload

# Update lock file after changing dependencies in pyproject.toml
uv lock
```

### Frontend

```bash
cd frontend
npm ci
npm run lint
npm run type-check
npm test
npm run build
```

## CI

Two GitHub Actions workflows (`.github/workflows/`):
- **backend.yml**: ruff check, ruff format --check, mypy, pytest with coverage (Python 3.12, uv)
- **frontend.yml**: eslint, tsc, vitest, vite build (Node 20)

Both trigger on pushes and PRs to main/master for their respective paths.

## Code Style

- Python >=3.12, line length 100
- Ruff for linting and formatting (rules: E, F, I, N, W, UP)
- Mypy strict mode with pydantic plugin
- pytest with asyncio_mode = "auto"
- Always run `uv run ruff check` and `uv run ruff format --check` before committing

## Security Scanning

Run CodeQL locally before pushing backend changes to catch security issues early (path traversal, injection, etc.):

```bash
# Install CodeQL (first time only)
cd /tmp && wget -q https://github.com/github/codeql-action/releases/latest/download/codeql-bundle-linux64.tar.gz -O codeql-bundle.tar.gz && tar -xzf codeql-bundle.tar.gz

# From project root — create database and analyze
/tmp/codeql/codeql database create /tmp/codeql-db --language=python --source-root . --overwrite
/tmp/codeql/codeql database analyze /tmp/codeql-db --format=sarif-latest --output=/tmp/codeql-results.sarif /tmp/codeql/qlpacks/codeql/python-queries/*/codeql-suites/python-security-extended.qls

# View results
python3 -c "
import json
results = json.load(open('/tmp/codeql-results.sarif'))['runs'][0]['results']
print(f'Findings: {len(results)}')
for r in results:
    locs = r.get('locations', [])
    for loc in locs:
        pl = loc.get('physicalLocation', {})
        f = pl.get('artifactLocation', {}).get('uri', '?')
        line = pl.get('region', {}).get('startLine', '?')
        print(f'  [{r[\"ruleId\"]}] {f}:{line}')
"
```

This matches the CodeQL checks that run on GitHub PRs. Fix any `py/path-injection` or other high-severity findings before pushing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fclinton) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
