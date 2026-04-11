## git-ai-commit

> - `cli.py`: Click-based entry; runs the end-to-end generator.

# Repository Guidelines

## Project Structure & Module Organization
- `cli.py`: Click-based entry; runs the end-to-end generator.
- `core/`: Pipeline and modules
  - `collectors/`, `llm/`, `formatter/`, `contracts/`, `registry.py`.
- `config/`: Default config (`default.yaml`), loaders, merge logic.
- `utils/`: Logging, git helpers, custom errors.
- `tests/`: Pytest/unittest tests (`tests/test_*.py`).

## Build, Test, and Development Commands
- Install (editable): `uv pip install -e .` or `pip install -e .`.
- Run CLI locally (no install): `uv run python cli.py --dry-run`.
- After install: `aicommit --dry-run`.
- Run tests: `uv run pytest -q` (or `pytest -q`).
- Lint/format (dev extras): `uv run black . && uv run isort .`.

## Coding Style & Naming Conventions
- Python 3.9+, 4-space indentation, PEP8.
- Formatting: Black; imports: isort; typing encouraged; mypy optional.
- Naming: functions/vars `snake_case`, classes `PascalCase`, constants `UPPER_SNAKE_CASE`.
- Keep modules decoupled (Collector/Provider/Formatter follow contracts); add docs for public APIs.

## Testing Guidelines
- Framework: pytest (some unittest-style classes). Files: `tests/test_*.py`.
- Run focused tests: `pytest tests/test_pipeline.py -q`.
- Mock external I/O (network, git) as in `tests/test_provider.py` and collectors’ tests.
- Prefer small unit tests per module; add golden tests for templates when changing formatting.

## Commit & Pull Request Guidelines
- Conventional Commits used (see history): e.g., `feat(collector): add IssueCollector`, `feat: implement Jinja2 formatter`.
- Scope is optional but preferred (`feat(cli): ...`).
- PRs: explain what/why/how, link issues (`Closes #123`), show sample output/screenshots, note config/template changes, and include tests.

## Security & Configuration Tips
- Set `OPENAI_API_KEY` in env; never commit secrets.
- Config precedence: default → user (`~/.aicommit/config.yaml`) → project (`.aicommit.yaml`) → CLI overrides.
- YAML supports env substitution: `api_key: !env ${OPENAI_API_KEY}`.
- Templates: default in `core/formatter/templates`; override via `formatter.template_dir` or `~/.aicommit/templates`.


# Language & Comment Standards
- **Code Comments**: All comments in scripts and configuration files must use Simplified Chinese.
- **Documentation**: README and help documentation may use mixed Chinese and English, but technical explanations should prioritize Simplified Chinese.
- **Variable Naming**: While variable names remain in English, related comments should use Chinese to explain their purpose and meaning.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/allen-mk)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/allen-mk)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
