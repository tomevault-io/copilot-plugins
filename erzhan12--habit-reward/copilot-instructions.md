## habit-reward

> Source lives in `src/`, split by responsibility: `bot/` holds Telegram handlers, `services/` contains framework-neutral logic, `airtable/` wraps external persistence, `dashboard/` powers Streamlit views, and `habit_reward_project/` mirrors the in-progress Django stack. Shared helpers sit in `src/utils/` plus `src/config.py`. Unit tests mirror this tree inside `tests/`, operational docs stay under `docs/`, while CLI helpers live in `commands/` and `scripts/`. Persisted data or generated assets belong in `staticfiles/` and `deployment/` holds docker-compose manifests plus automation scripts.

# Repository Guidelines

## Project Structure & Module Organization
Source lives in `src/`, split by responsibility: `bot/` holds Telegram handlers, `services/` contains framework-neutral logic, `airtable/` wraps external persistence, `dashboard/` powers Streamlit views, and `habit_reward_project/` mirrors the in-progress Django stack. Shared helpers sit in `src/utils/` plus `src/config.py`. Unit tests mirror this tree inside `tests/`, operational docs stay under `docs/`, while CLI helpers live in `commands/` and `scripts/`. Persisted data or generated assets belong in `staticfiles/` and `deployment/` holds docker-compose manifests plus automation scripts.

## Build, Test, and Development Commands
Run `make sync` once per environment to install dependencies through `uv`. `make bot` executes `uv run python -m src.bot.main` for interactive development, and `make dashboard` starts the Streamlit UI. Use `make lint`, `make format`, and `make check` to run Ruff in lint-only, format, or full validation modes. Quick smoke scripts `./run_bot.sh` and `./run_dashboard.sh` mirror the Make targets when you need minimal bootstrapping.

## Coding Style & Naming Conventions
Target Python 3.13 with PEP 8 spacing (4 spaces, 100-char soft wrap). Favor descriptive snake_case for modules, functions, and variables; constants stay upper snake case. All user-facing strings originate from `src/bot/messages.py`, and bot handlers must follow the validation flow documented in `RULES.md`. Keep side effects isolated to `src/bot/`, leaving `src/services/` portable. Apply type hints on new or modified code paths.

## Testing Guidelines
Pytest with `pytest-asyncio` drives the suite. Co-locate tests with their subject module using the `tests/<package>/test_<feature>.py` naming pattern. Run `uv run pytest tests/` locally before opening a PR, and prefer `uv run pytest --cov=src tests/` to satisfy existing coverage gates. Mock Airtable or Telegram dependencies via fixtures instead of hitting live services.

## Commit & Pull Request Guidelines
Commits use imperative Conventional Commit subjects (`fix: handle reward rollover`). Keep messages under ~72 characters and describe the why in the body when needed. PRs should outline business context, enumerate validation steps (e.g., `make check`, `uv run pytest --cov=src tests/`), link related issues or docs, and attach screenshots or bot transcript snippets for UI or conversational updates.

## Security & Configuration Tips
Never commit secrets; start from `.env.example` and store live credentials only in `.env` or CI secrets. Review `src/config.py` before adding new environment flags, and keep TELEGRAM tokens, API keys, and database URLs in GitHub Actions or server secret stores. When touching auth or messaging flows, cross-check `RULES.md` to preserve locale coverage and validation gating.

---
> Source: [erzhan12/habit-reward](https://github.com/erzhan12/habit-reward) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
