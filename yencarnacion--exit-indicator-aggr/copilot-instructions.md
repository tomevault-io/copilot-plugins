## exit-indicator-aggr

> - `server_py/` holds the FastAPI business logic: `app.py` wires routes, `state.py` tracks order book state, `depth.py` and `ib_client.py` wrap Interactive Brokers data. Config defaults live in `config.py` and `config.tws.yaml`.

# Repository Guidelines

## Project Structure & Module Organization
- `server_py/` holds the FastAPI business logic: `app.py` wires routes, `state.py` tracks order book state, `depth.py` and `ib_client.py` wrap Interactive Brokers data. Config defaults live in `config.py` and `config.tws.yaml`.
- `tests/` mirrors server modules with pytest suites; add new fixtures in `conftest.py`.
- `web/` serves the lightweight client (`app.js`, `styles.css`, `sounds/`) that consumes the websocket feed.
- `config-data/` and root YAML files store environment-specific settings; avoid committing credentials.

## Build, Test, and Development Commands
- `python -m venv .venv && . .venv/bin/activate` — create/enter the local virtualenv.
- `pip install -r server_py/requirements-dev.txt` — sync runtime plus dev tooling (pytest, typer, etc.).
- `./go.sh` — boot the auto-reloading uvicorn server using `server_py/run.sh` and the selected YAML config.
- `PYTHONPATH=. uvicorn server_py.app:app --reload` — manual launch when you need custom flags.
- `pytest` — execute the full asynchronous test suite defined in `pytest.ini`.

## Coding Style & Naming Conventions
- Follow PEP 8: 4-space indentation, descriptive snake_case for Python, PascalCase for classes, SCREAMING_SNAKE_CASE for constants.
- Prefer type hints on new Python functions; keep public API signatures annotated.
- Keep module-level configuration isolated in `config.py`; pass dependencies explicitly rather than using globals.
- For web assets, stick to ES modules and camelCase for functions; keep sounds/CSS filenames lowercase with hyphens.

## Testing Guidelines
- Write pytest unit tests under `tests/` mirroring the module path, e.g., `server_py/depth.py` → `tests/test_depth.py`.
- Use descriptive test names like `test_depth_handles_empty_book` and favor fixtures in `conftest.py`.
- Ensure new websocket or state mutations include regression tests covering edge cases (empty books, reconnects).
- Aim to maintain or raise current coverage; add async tests when touching coroutine logic.

## Commit & Pull Request Guidelines
- Use short, imperative commit messages (`Add DOM snapshot endpoint`) and group related changes.
- Reference relevant issue IDs in the body when applicable; summarize config changes explicitly.
- For PRs: include a concise problem/solution description, test evidence (`pytest` output, screenshots for UI tweaks), and call out config or deployment impacts.
- Request review before merging; keep diffs focused and rebased for a linear history.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yencarnacion)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yencarnacion)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
