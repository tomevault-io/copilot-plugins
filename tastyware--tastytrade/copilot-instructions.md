## tastytrade

> - Install UV: `curl -LsSf https://astral.sh/uv/install.sh | sh`

## Environment Setup
- Python 3.11+
- Install UV: `curl -LsSf https://astral.sh/uv/install.sh | sh`
- Create environment and install dependencies: `uv sync`
- Activate environment: `source .venv/bin/activate`

## Commands

### File-scoped (preferred - fast feedback)
uv run ruff check tastytrade/changed_file.py
uv run pyright tastytrade/changed_file.py
uv run pytest -svk 'my_test_name'

### Full suite (only when explicitly requested/before opening a PR)
See @Makefile (do `make lint` and `make test` if `make` is available; otherwise do the individual commands listed there)

## Testing
Make sure you have `TT_REFRESH`, `TT_SECRET`, and `TT_ACCOUNT` environment variables set--tests require hitting the Tastytrade API. If users aren't comfortable doing this, they can open a PR anyways and tests will run with `pull_request_target` upon approval.

## Project Structure
- `docs/` - Main source for project information and implementation examples; prefer this for up-to-date code as SDK changes frequently.
- `tastytrade/` - SDK implementation lives here
- `tests/` - pytest tests live here

## Permissions

### Allowed without prompting
- Read files, list directories
- Single file linting, type checking, formatting

### Require approval first
- Any trades that pass `dry_run=False` (these are real trades that will hit the API!)
- Unit tests on specific files
- Package installations (`uv add`, `uv pip install` for temporary use is fine)
- Git operations (`git push`, `git commit`)
- File deletion
- Running full lint/test suites

---
> Source: [tastyware/tastytrade](https://github.com/tastyware/tastytrade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
