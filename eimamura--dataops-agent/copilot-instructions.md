## dataops-agent

> - Runtime package lives under `src/aiagent/` with `agent.py` (LangChain agent logic), `tools.py` (custom BaseTool implementations), `config.py` (Pydantic `Settings`), and `cli.py` (Typer entrypoint).

# Repository Guidelines

## Project Structure & Module Organization
- Runtime package lives under `src/aiagent/` with `agent.py` (LangChain agent logic), `tools.py` (custom BaseTool implementations), `config.py` (Pydantic `Settings`), and `cli.py` (Typer entrypoint).
- Environment samples and Docker assets sit at the repo root (`.env.example`, `compose.yaml`), while infrastructure bootstrap SQL lives in `infra/postgres/`.
- Tests are not yet scaffolded; add new suites under `tests/` using the src-layout import path (`from aiagent import ...`).

## Build, Test, and Development Commands
- `uv venv && source .venv/bin/activate`: create and enter the recommended virtual env (plain `python -m venv .venv` also works).
- `uv pip install -e ".[dev]"`: install runtime plus dev extras (LangChain, Typer, Ruff, pytest).
- `python -m aiagent.cli "ping the warehouse"`: run the CLI one-off; use `--help` for interactive flags.
- `docker compose up -d postgres`: launch the optional Postgres 16 dependency defined in `compose.yaml`.

## Coding Style & Naming Conventions
- Target Python 3.11 with type hints, descriptive function names, and LangChain-friendly abstractions.
- Format/lint via Ruff (`ruff check .`) with 100-char lines; keep modules ASCII unless an external API demands otherwise.
- Settings flow through helpers like `get_settings()`; prefer `settings.model_copy(update=...)` for overrides instead of mutating globals.
- Name tools as `<Domain>Tool` (e.g., `SQLQueryTool`) and Typer commands with hyphenated verbs (`sync-data`).

## Testing Guidelines
- Use `pytest` with feature-focused modules (e.g., `tests/test_agent_routing.py`); fixtures belong in `tests/conftest.py`.
- Mirror feature names in test files (`test_cli.py`, `test_tools_sql.py`) and assert both LangChain behavior and CLI UX.
- Aim for meaningful coverage of new tools (happy path + error guardrails); document heavy integration steps in test docstrings.

## Commit & Pull Request Guidelines
- Follow the existing short, imperative commit style (`add sql utils`, `update readme`). Scope one logical change per commit.
- PRs should call out: purpose, primary commands run (`pytest`, `ruff check .`), config updates, and any screenshots/logs relevant to CLI changes.
- Link tracking issues or TODOs in the PR body and note database or environment prerequisites (e.g., needing `.secrets/postgres_password`).

## Security & Configuration Tips
- Copy `.env.example` to `.env`, fill `LLM_PROVIDER`, Hugging Face/OpenAI keys, and optional `DATABASE_URL`. Never commit `.env` or `.secrets/*`.
- Rotate credentials stored in `.secrets/postgres_password` and prefer local `.env` overrides instead of editing default settings in code.
- When sharing agent demos, scrub transcripts that might include customer data or SQL query outputs.

---
> Source: [eimamura/dataops-agent](https://github.com/eimamura/dataops-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
