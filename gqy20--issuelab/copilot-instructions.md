## issuelab

> - `src/issuelab/`: core Python package (CLI entry points, agent execution, config, GitHub tooling).

# Repository Guidelines

## Project Structure & Module Organization
- `src/issuelab/`: core Python package (CLI entry points, agent execution, config, GitHub tooling).
- `tests/`: pytest suite (unit and behavior coverage for parsing and agent orchestration).
- `agents/`: per-user agent configs and prompts (e.g., `agents/<name>/agent.yml`, `agents/<name>/prompt.md`).
- `scripts/`: lightweight wrappers for GitHub Actions (minimal deps, no full install).
- `docs/`: [architecture, deployment, and setup guides](./docs/).
- `config/`: workflow and runtime configuration assets.

## Build, Test, and Development Commands
- `uv sync`: install dependencies for local development.
- `uv run pytest tests/ --cov=issuelab`: run the full test suite with coverage.
- `uv run pytest tests/test_parser.py -v`: run a focused test file.
- `uv run ruff check src/ tests/`: lint (imports, style, common errors).
- `uv run ruff format --check src/ tests/`: verify formatting.
- `uv run mypy src/issuelab --ignore-missing-imports`: static type checks.
- `uv build`: build the package.
- Example CLI: `uv run python -m issuelab review --issue 1 --post`.

## Coding Style & Naming Conventions
- Python 3.13+, 4-space indentation, line length 120.
- Formatting and linting via Ruff (double quotes preferred, import sorting enabled).
- Module names are `snake_case`; classes are `PascalCase`; functions are `snake_case`.
- Keep prompts and agent configs named by role or GitHub handle.

## Testing Guidelines
- Frameworks: `pytest`, `pytest-asyncio`, `pytest-cov`.
- Test files follow `tests/test_*.py` naming.
- Use `uv run pytest ...` to ensure the correct environment.

## Commit & Pull Request Guidelines
- Commit history follows Conventional Commits: `feat:`, `fix:`, `chore:`, `docs:`, `test:`.
- PRs should include a clear summary, linked issue (if any), and relevant workflow output or screenshots when behavior changes.
- Keep PRs focused and avoid mixing unrelated refactors with feature work.

## Security & Configuration Tips
- Secrets should be provided via GitHub Actions (e.g., `ANTHROPIC_AUTH_TOKEN`, `PAT_TOKEN`).
- Use GitHub App credentials where possible for cross-repo dispatch; see [docs/DEPLOYMENT.md](./docs/DEPLOYMENT.md).
- MCP configuration lives in `.mcp.json` (global) and `agents/<name>/.mcp.json` (per-agent overrides).

---
> Source: [gqy20/IssueLab](https://github.com/gqy20/IssueLab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
