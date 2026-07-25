## dropmcp

> - **Don't assume.** Don't hide confusion. Surface tradeoffs and ask before guessing.

# AGENTS.md

## Behavioural norms

- **Don't assume.** Don't hide confusion. Surface tradeoffs and ask before guessing.
- **Simplicity first.** Minimum code that solves the problem. Nothing speculative.
- **Surgical changes.** Touch only what you must. Clean up only your own mess.
- **Goal-driven execution.** Define success criteria up front. Loop until verified.

## Project overview

`dropmcp` is a Python library that turns a `skills/` and `prompts/` folder into a
hosted [FastMCP](https://gofastmcp.com) server with a browseable React catalog UI.
Stack: Python (FastMCP, SQLAlchemy, PyYAML) for the server, React + TypeScript +
Vite for the catalog SPA. It exists to serve skills/prompts to remote MCP clients
over streamable-HTTP, replacing the per-server boilerplate with one reusable engine.

## Project layout

| Path | What |
|---|---|
| `pyproject.toml` | package metadata, deps, ruff/pytest config (hatchling build) |
| `src/dropmcp/` | the library — `server.py` (factory), `config.py`, `skills.py`, `prompts.py`, `catalog.py`, `feedback.py`, `subscriptions.py`, `telemetry.py`, `validate.py` |
| `src/dropmcp/static/dist/` | **built** SPA, packaged into the wheel (do not edit by hand) |
| `client/` | React/TS catalog SPA (Vite); `client/tests/` are Playwright tests |
| `tests/` | pytest suite (`test_*.py`, `conftest.py`) |
| `examples/` | runnable `server.py` + sample skills/prompts |
| `template/` | copier scaffold for generating new servers |
| `.github/workflows/ci.yml` | CI: python tests, UI tests, wheel build, PyPI publish |

## Build / contribute

```bash
# Python
pip install -e ".[dev,otel]"
ruff check src        # lint (line-length 88)
pytest                # tests/ ; asyncio_mode=auto

# Catalog UI (in client/)
cd client
npm ci
npm run build         # outputs to ../src/dropmcp/static/dist
npm test              # Playwright (needs: npx playwright install --with-deps chromium)
npm run lint          # eslint
```

Before pushing: run `ruff check src`, `pytest`, and (if you touched `client/`)
`npm run build` + `npm test`.

## Links

- README — install, configuration table, hosting, skill/prompt format: [`README.md`](README.md)
- FastMCP docs: https://gofastmcp.com
- PyPI: https://pypi.org/project/dropmcp/

## Important notes

- **The SPA must be built before the wheel is packaged.** The wheel force-includes
  `src/dropmcp/static/dist`; an unbuilt SPA ships a broken catalog. CI builds it
  for every job — do the same locally before testing packaging.
- **Releases** are cut by pushing a `v*` git tag. The version in `pyproject.toml`
  and `src/dropmcp/__init__.py` (`__version__`) **and** the tag must all match.
- **Tests must be deterministic** — no retries; fix the root cause.
- **Only test code in this repo.** Don't write tests for third-party/library behaviour.
- **Eval results & StarRocks are optional** — keep that coupling behind the
  `[starrocks]` extra and the pluggable store; don't pull it into the base package.
- Server is streamable-HTTP only (hosted, multi-client) — there is no local stdio transport.

---
> Source: [agoda-com/dropmcp](https://github.com/agoda-com/dropmcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
