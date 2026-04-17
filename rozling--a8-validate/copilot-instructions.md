## a8-validate

> - Always use the virtual environment at `venv/` or `.venv/` when available (use whichever exists)

# Cursor Rules for a8-validate

## Python Environment
- Always use the virtual environment at `venv/` or `.venv/` when available (use whichever exists)
- Prefer `venv/bin/python` or `.venv/bin/python` for Python execution
- When running commands, activate the virtual environment first or use `venv/bin/python -m <command>` (or `.venv/bin/python -m <command>`)
- For pip installs, use `venv/bin/pip` or `.venv/bin/pip` (or `python -m pip install` from within the venv)

## Code Style
- Follow PEP 8 style guidelines
- Use black for code formatting (120 character line length)
- Use isort for import sorting (black-compatible profile)
- Maximum line length: 120 characters
- Run flake8 before committing (with max-line-length=120, extend-ignore=E203,W503,D)

## Testing
- Run tests using `pytest` from the virtual environment
- Test command: `venv/bin/pytest -v --tb=short` or `.venv/bin/pytest -v --tb=short` (or `python -m pytest -v --tb=short`)
- All tests are located in `a8_validate/tests/`
- Ensure the full pytest suite passes (56 tests at last count; run `pytest --collect-only -q` to verify)

## Dependencies
- Production dependencies are in `requirements.txt`
- Development dependencies are in `requirements-dev.txt`
- Never add `wave` to requirements.txt (it's a standard library module)
- Always use `venv/bin/pip` or `.venv/bin/pip` for installing dependencies

## File Structure
- Main validation code: `a8_validate/`
- CLI script: `validate_directory.py`
- Utility scripts: `scripts/`
- Tests: `a8_validate/tests/`

## Git Workflow
- Run pre-commit hooks before committing
- Ensure all CI/CD checks pass (Tests, Lint, Coverage, Security)

## Code Quality
- Remove unused imports
- Remove unused variables
- Keep functions focused and single-purpose
- Add docstrings for public functions and classes
- Use type hints where appropriate

### API / validation
- When adding optional “don’t mutate” (or similar) behavior, keep backward compatibility: add a parameter with a default that preserves current behavior, document both modes and return value, and test both branches (input unchanged when non-mutating; same object when mutating).
- For “validate this file/directory” flows, prefer not mutating parsed data (e.g. use `mutate=False` and pass the returned normalized copy to downstream validators); reserve in-place mutation for explicit “normalize and write back” flows.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rozling) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
