## stock-forecast

> Core coding standards and project conventions for stock-forecast


# Stock Forecast Standards

## Environment

- Activate virtual environment per README.md (e.g., `uv run` or `source .venv/bin/activate`)
- Use dependencies from `pyproject.toml`

## Code Style

- Follow PEP 8
- Use descriptive variable names reflecting the data
- Prefer functional programming; avoid unnecessary classes
- **Enhance in place**: When improving a method, update the existing name—don't create new ones

## Project Layout

```
src/data_preprocess/   → data loading, cleaning, feature engineering
src/forecasting/       → pipeline, models, backtesting
src/feature_engineering/ → feature utilities
notebooks/             → development and testing
data/                  → CSV/parquet storage
config/                → YAML config
tests/                 → unit tests
```

## Key Conventions

1. Define reusable code in `src/`; import into notebooks
2. Begin analysis with data exploration and summary statistics
3. Document data sources, assumptions, and methodologies
4. Add unit tests for new functionality

## Orchestration

Multi-step or cross-domain work (forecasting, UI, tests, git in one effort) should use **`.cursor/skills/agent-manager/`**: clarify ambiguous requirements with the user, plan execution, then delegate to specialist skills (`stock-forecast-analysis`, `ui-agent`, `testing-agent`, `git-agent` as appropriate). See `SKILL.md` in that folder.

## Git workflow

- **Do not commit or edit project code directly on `main`**. `main` is for merged, reviewed work only.
- **Use a feature branch** for any code changes: sync `main` (`git pull`), then `git checkout -b <descriptive-branch-name>` before editing.
- Details (commit style, tests before push, PRs): `.cursor/skills/git-agent/`.

## Agent & Safety Conventions

- **Secrets**: Never commit API keys, tokens, or real credentials. Use environment variables (e.g. `STOCK_DATA_DIR`, `.env` gitignored); do not paste secrets into chat or code.
- **Scope**: Implement what was asked; avoid unrelated refactors or extra files unless the user requested them.
- **Destructive actions**: Confirm with the user before irreversible steps (e.g. `git reset --hard`, mass deletes, overwriting shared data).
- **Read before edit**: Match existing naming, imports, and patterns in the touched files.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/StephenHB) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
