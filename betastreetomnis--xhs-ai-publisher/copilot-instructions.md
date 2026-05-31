## xhs-ai-publisher

> - `main.py`: app entrypoint.

# Repository Guidelines

## Project Structure & Module Organization

- `main.py`: app entrypoint.
- `src/`: primary source code.
  - `src/core/pages/`: PyQt UI pages (home/settings/tools, etc.).
  - `src/core/services/`: business services (LLM, template rendering, Chrome profile, etc.).
  - `src/core/processor/`: content/image processing threads and helpers.
- `templates/`: prompt templates (e.g. `templates/prompts/*.json`).
- `assets/`: bundled template showcase assets.
- `tests/`: unit/integration tests and pytest config (`tests/pytest.ini`).
- `docs/`, `images/`: documentation and screenshots.

## Build, Test, and Development Commands

Install & run (macOS/Linux):

```bash
./install.sh
./启动程序.sh
```

Manual run:

```bash
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python main.py
```

Playwright browser (only if needed):

```bash
PLAYWRIGHT_BROWSERS_PATH="$HOME/.xhs_system/ms-playwright" python -m playwright install chromium
```

Tests:

```bash
pip install -r tests/requirements.txt
pytest -q
pytest -m unit
```

## Coding Style & Naming Conventions

- Python: 4-space indentation, `snake_case` for functions/vars, `CapWords` for classes.
- Keep UI code in `src/core/pages/` and reusable logic in `src/core/services/`.
- Prefer small, testable functions; avoid unrelated refactors in the same PR.

## Testing Guidelines

- Framework: `pytest` (see markers in `tests/pytest.ini` like `unit`, `integration`, `browser`).
- Naming: `test_*.py`, `Test*` classes, `test_*` functions.
- Mark slow/network/browser tests appropriately and keep unit tests deterministic.

## Commit & Pull Request Guidelines

- Commit style in this repo commonly uses prefixes like `feat:`, `fix:`, `docs:`, `chore:` (Chinese descriptions are OK).
- PRs should include: a short problem/solution summary, steps to verify, and UI screenshots when changing layouts (store under `docs/assets/` or `images/`).
- If you change user-facing behavior, update both `readme.md` and `readme_en.md`.

## Security & Configuration Tips

- Never commit real keys. Use `.env` locally (it’s gitignored) and keep secrets empty in `.env.example`.
- Local runtime data lives under `~/.xhs_system/` (db, logs, cached images, Playwright browsers).

---
> Source: [BetaStreetOmnis/xhs_ai_publisher](https://github.com/BetaStreetOmnis/xhs_ai_publisher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
