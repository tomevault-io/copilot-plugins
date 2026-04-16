## skills

> You are an AI assistant working in the Qredence Skills monorepo. Please adhere to the following rules at all times.

# AI Agent Rules for Qredence Skills Monorepo

You are an AI assistant working in the Qredence Skills monorepo. Please adhere to the following rules at all times.

## Python and Dependency Management
- **ALWAYS use `uv`** as the primary Python package and environment manager.
- DO NOT use `pip`, `poetry`, `pipenv`, or `venv` directly unless explicitly required by a legacy script that cannot be updated.
- Examples:
  - To run a script: `uv run python scripts/script_name.py`
  - To install a dependency: `uv add <package>`
  - To sync dev dependencies: `uv sync --dev`

## Code Quality
- **ALWAYS use `ruff`** for linting and formatting.
- Example: `uv run ruff check .` and `uv run ruff format .`
- Do not use `flake8`, `black`, `isort`, or `autopep8`.

## Skill Creation
- When asked to create a new skill, ALWAYS use the scaffolding script first to ensure the correct directory structure:
  `uv run python scripts/init_skill.py <skill-name> --path skills/`
- Every skill MUST have a `SKILL.md` file at its root.
- To package and validate a skill, use: `uv run python scripts/package_skill.py skills/<skill-name>`

## Monorepo Context
- The repository contains multiple independent skills under the `skills/` directory.
- Ensure any global changes (like updating CI or root scripts) do not inadvertently break individual skills.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Qredence) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
