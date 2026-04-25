## hegelion

> - `hegelion/`: core Python package (dialectic logic, MCP server, scripts, training utilities).

# Repository Guidelines

## Project Structure & Module Organization
- `hegelion/`: core Python package (dialectic logic, MCP server, scripts, training utilities).
- `tests/`: pytest suite (files named `test_*.py`).
- `docs/`: specs and guides, including `docs/HEGELION_SPEC.md` and MCP/agent docs.
- `scripts/`: release and maintenance scripts (e.g., PyPI publishing).
- `hegelion-mcp-node/`: Node/TypeScript MCP server for non-Python environments.
- `hegelion-web/`, `apps/`, `extensions/`, `examples/`, `benchmarks/`, `hegelion-data/`: auxiliary apps, docs, and datasets.

## Build, Test, and Development Commands
- Python deps (recommended): `uv sync --dev`
- Python deps (pip): `pip install -e ".[dev]"`
- Run Python tests: `uv run pytest -v`
- Coverage: `uv run pytest --cov=hegelion --cov-report=html`
- Format/lint: `uv run black hegelion tests` and `uv run ruff check hegelion tests`
- Run MCP server: `python -m hegelion.mcp.server` (or `hegelion-server` when installed)
- Node MCP server (from `hegelion-mcp-node/`): `npm install`, `npm run build`, `npm start`

## Coding Style & Naming Conventions
- Python formatted with Black (line length 100) and linted with Ruff.
- Prefer type hints on public functions and Google-style docstrings for public APIs.
- Tests live in `tests/` and follow `test_*.py` naming.
- Branch names: `feature/...`, `fix/...`, `docs/...`, `refactor/...`.

## Testing Guidelines
- Primary framework is `pytest` with `pytest-asyncio`.
- New features should include tests that mirror package structure in `tests/`.
- If you touch `hegelion-mcp-node/`, validate with `npm run build` (no test script is defined).

## Commit & Pull Request Guidelines
- Use Conventional Commit-style subjects seen in history, e.g. `feat: ...`, `fix(mcp): ...`, `docs: ...`, `chore: ...`, `style: ...`, `refactor: ...`.
- PRs should include a clear summary, linked issues where relevant, and notes on tests run.
- Update `README.md` and `docs/HEGELION_SPEC.md` when behavior or APIs change; include screenshots for `hegelion-web/` UI changes.

## Security & Configuration Tips
- Use `.env.example` to configure API keys; do not commit `.env`.
- Keep secrets in environment variables for local runs and MCP client configs.

## Agent-Specific References
- Dialectical agent loop and MCP prompt workflows: `docs/guides/agents.md`.
- Training workflow details: `hegelion/training/AGENT.md`.

---
> Source: [Hmbown/Hegelion](https://github.com/Hmbown/Hegelion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
