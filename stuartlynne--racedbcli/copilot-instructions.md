## racedbcli

> - `cli/`: End-user CLI scripts (e.g., `participants.py`, `licenseholders.py`, `numberset.py`, `competition.py`). Execute directly with Python.

# Repository Guidelines

## Project Structure & Module Organization
- `cli/`: End-user CLI scripts (e.g., `participants.py`, `licenseholders.py`, `numberset.py`, `competition.py`). Execute directly with Python.
- `libs/`: Shared helpers (e.g., `racedb.py`, `racedbsql.py`, `download.py`, `upload.py`). Import from CLI tools.
- `tests/`: Lightweight scripts demonstrating behavior (`testbase.py`, `testsextend.py`). Currently not using a test runner.
- Root assets: Example spreadsheets and HTML used during workflows.

## Build, Test, and Development Commands
- Create venv: `python -m venv .venv && source .venv/bin/activate`
- Install runtime deps: `pip install requests openpyxl pandas click` (plus `beautifulsoup4` if needed).
- Run a tool: `python cli/participants.py --host http://localhost:8080 --date 2025-06-01 --name "Event Name"`
- Other tools: `python cli/licenseholders.py ...`, `python cli/prereg.py ...`, `python cli/competition.py ...`
- Quick tests (ad hoc): `python tests/testbase.py` or `python tests/testsextend.py`

## Coding Style & Naming Conventions
- Python 3.8+; use 4-space indentation, PEP 8 naming (`snake_case` for functions/vars, `CapWords` for classes, modules lowercase).
- Keep CLI entry files small; move logic into `libs/` for testability.
- Prefer explicit imports from `libs` (e.g., `from libs.racedb import RaceDB`).
- Optional tooling: `pip install black ruff` then format with `black .` and lint with `ruff .`.

## Testing Guidelines
- Place new tests in `tests/` and name `test_*.py`.
- Favor pure functions in `libs/` and exercise them in tests; avoid network calls by mocking or isolating I/O.
- If introducing pytest, standardize on `pytest -q` and add minimal fixtures; otherwise keep current ad-hoc scripts.

## Commit & Pull Request Guidelines
- Commits: imperative mood, concise subject (≤ 72 chars), body explaining context and rationale. Group related changes.
- PRs: clear description, reproduction steps (commands and sample args), expected vs. actual behavior, and screenshots/CLI output when useful. Link related issues.
- Keep diffs focused; update README or inline help (`--help`) when changing CLI behavior.

## Security & Configuration Tips
- Do not commit secrets. Tools may read `RACEDB_USERNAME`/`RACEDB_PASSWORD`; prefer env vars over hardcoding.
- Use local/test hosts (e.g., `http://localhost:8080`) when developing. Be careful with real data exports in the repo.

---
> Source: [stuartlynne/racedbcli](https://github.com/stuartlynne/racedbcli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
