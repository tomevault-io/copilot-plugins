## banana-straightener

> Source lives in `src/banana_straightener/`: `agent.py` holds iteration logic, `models.py` wraps Gemini APIs, `cli.py` exposes the `straighten` entry points, `ui.py` runs the Gradio interface, and `utils.py` provides shared helpers. Tests sit in `tests/` with quick, integration, and image-generation suites. Example prompts land in `examples/`, run artifacts go to `outputs/`, and release helpers reside in `scripts/`. Packaging is managed via `pyproject.toml`, with pytest config in `pytest.ini` and environment templates in `.env.example`.

# Repository Guidelines

## Project Structure & Module Organization
Source lives in `src/banana_straightener/`: `agent.py` holds iteration logic, `models.py` wraps Gemini APIs, `cli.py` exposes the `straighten` entry points, `ui.py` runs the Gradio interface, and `utils.py` provides shared helpers. Tests sit in `tests/` with quick, integration, and image-generation suites. Example prompts land in `examples/`, run artifacts go to `outputs/`, and release helpers reside in `scripts/`. Packaging is managed via `pyproject.toml`, with pytest config in `pytest.ini` and environment templates in `.env.example`.

## Build, Test, and Development Commands
- `uv venv && source .venv/bin/activate`: create and enter a local virtualenv.
- `uv pip install -e .[dev]`: editable install with dev extras.
- `straighten --help` / `straighten generate "a perfectly straight banana"`: inspect CLI usage or run a single-image generation.
- `straighten generate "blend styles" -i style1.png -i style2.jpg`: demonstrate multi-image conditioning.
- `straighten ui`: launch the Gradio UI in a browser.

## Coding Style & Naming Conventions
Target Python 3.12 (supports 3.10+). Use 4-space indents, `snake_case` for functions/variables, `PascalCase` for classes, and constants in `UPPER_SNAKE_CASE`. Format with `uv run black src/ tests/` (line length 88). Lint via `uv run flake8 src/ tests/`, and validate typing with `uv run mypy src/banana_straightener/`. Public APIs should include type hints and concise docstrings where behavior is non-obvious.

## Testing Guidelines
Pytest drives the suite (`pytest.ini` already scopes to `test_*.py`). Run smoke tests with `uv run pytest tests/test_quick.py -v`; full coverage with `uv run pytest --cov=banana_straightener --cov-report=term-missing`. Provide deterministic fixtures, mock external Gemini calls when practical, and skip or mark slow/API-dependent scenarios. Add new cases under `tests/` following `test_<feature>.py`.

## Commit & Pull Request Guidelines
Write commits in imperative mood (e.g., “Add multi-image blending”). Group related changes and keep diff noise low. Pull requests should explain motivation, summarize behavior changes, and link issues. Include CLI snippets, screenshots, or sample outputs if UX changes. Ensure tests, lint, type checks, and formatting pass before requesting review.

## Security & Configuration Tips
Set `GEMINI_API_KEY` via `.env` or shell; never commit secrets. Store temporary assets in `outputs/`. Clean up large generated files before pushing, and confirm third-party assets are licensed for distribution.

---
> Source: [velvet-shark/banana-straightener](https://github.com/velvet-shark/banana-straightener) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
