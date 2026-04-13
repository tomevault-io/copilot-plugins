## test-target-calc

> Application code lives under `src/pycalc/`. Use `engine.py` for shared calculation logic, `cli.py` for command-line behavior, and `web/` for the Flask app entrypoints. HTML templates are stored in `src/pycalc/templates/`. Tests live in `tests/` and mirror user-facing areas such as engine, CLI, web, and cross-surface consistency. Utility scripts are in `scripts/`. CI workflows are defined in `.github/workflows/`.

# Repository Guidelines

## Project Structure & Module Organization

Application code lives under `src/pycalc/`. Use `engine.py` for shared calculation logic, `cli.py` for command-line behavior, and `web/` for the Flask app entrypoints. HTML templates are stored in `src/pycalc/templates/`. Tests live in `tests/` and mirror user-facing areas such as engine, CLI, web, and cross-surface consistency. Utility scripts are in `scripts/`. CI workflows are defined in `.github/workflows/`.

## Build, Test, and Development Commands

Create a local environment and install tooling:

```powershell
python -m venv .venv
.venv\Scripts\python -m pip install -U pip
.venv\Scripts\python -m pip install -e .[dev]
```

Run the full local quality gate:

```powershell
python scripts/check.py
```

Run individual tasks when needed:

```powershell
python -m pytest
python -m ruff check .
python -m build
python scripts/package_web.py
python -m pycalc.cli "1 + 2 * 3"
python -m pycalc.web.app
```

## Coding Style & Naming Conventions

Follow Python conventions: 4-space indentation, `snake_case` for functions and modules, `PascalCase` for classes, and short, explicit names for error codes and helper functions. Keep business logic in the shared engine instead of duplicating it in CLI or web layers. Use ASCII unless a file already requires Unicode. Ruff is the active linting tool; keep code clean enough to pass `python -m ruff check .`.

## Testing Guidelines

Pytest is the test framework. Add tests beside the affected behavior in files named `test_*.py`. Prefer focused tests that cover both successful calculations and failure cases such as invalid syntax, unsupported tokens, and division by zero. Any surface change should preserve consistency across engine, CLI, and web flows.

## Commit & Pull Request Guidelines

Match the existing commit style: `<type>: <summary>`, for example `feat: add flask web calculator` or `test: add quality gate coverage`. Keep commits scoped to one logical change. Before opening a pull request, run `python scripts/check.py` and include a short description of the user-visible change, validation performed, and any packaging or workflow impact. Add screenshots only when the web UI changes materially.

## Security & Configuration Tips

Do not use `eval` or other arbitrary execution paths for expression handling; route all calculation work through `src/pycalc/engine.py`. For web runs, prefer setting `PYCALC_SECRET_KEY` instead of relying on the development fallback.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/baoblackcoal)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/baoblackcoal)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
