## loushang

> This repository uses a `src/` layout. Core package code lives in `src/loushang/`, with AI-facing modules under `src/loushang/ai/`, agent code under `src/loushang/agent/`, and coding/session infrastructure under `src/loushang/coding/`. Tests live in `tests/`. Runnable examples live in `examples/ai/`, with supporting scripts in `scripts/` and exploratory work in `spikes/`. Architecture and design notes belong in `docs/`.

# Repository Guidelines

## Project Structure & Module Organization
This repository uses a `src/` layout. Core package code lives in `src/loushang/`, with AI-facing modules under `src/loushang/ai/`, agent code under `src/loushang/agent/`, and coding/session infrastructure under `src/loushang/coding/`. Tests live in `tests/`. Runnable examples live in `examples/ai/`, with supporting scripts in `scripts/` and exploratory work in `spikes/`. Architecture and design notes belong in `docs/`.

## Build, Test, and Development Commands
Prefer `uv` for all Python workflows in this repository.

- `make bootstrap` — create `.venv` and install the package with dev dependencies.
- `uv run pytest tests -q` — run the full test suite.
- `make test-ai` — run AI-focused tests.
- `make check-ai` — run AI lint, typecheck, and focused tests; run before committing or pushing AI changes, and fix/report any failures.
- `make lint-ai` — run `ruff check` on AI modules and tests.
- `make fmt-ai` — format AI modules and tests with Ruff.
- `make typecheck-ai` — run `mypy` on `src/loushang/ai`.
- `make example-ai-complete` — run a representative example.

Use `uv run python ...` instead of bare `python ...`, and `uv pip ...` instead of `pip ...`.

## Coding Style & Naming Conventions
Target Python 3.11+ and use 4-space indentation. Follow existing naming patterns: modules and files in `snake_case`, classes in `PascalCase`, functions and variables in `snake_case`, constants in `UPPER_SNAKE_CASE`. Keep examples readable and user-facing: prefer the public `loushang.ai` API over internal provider wiring unless a file is explicitly labeled advanced.

## Testing Guidelines
Tests use `pytest` with `--import-mode=importlib` and `src` on `PYTHONPATH`. Name files `test_*.py` and keep test scope narrow and behavior-focused. Add or update tests when changing provider behavior, model resolution, auth handling, or public examples. Run targeted tests first, then broader suites if the change crosses subsystem boundaries.

## Commit & Pull Request Guidelines
Recent history uses short summaries and occasional conventional prefixes such as `test(integration): ...`. Prefer concise, imperative commit messages; use `type(scope): summary` when helpful. PRs should explain the user-visible change, list validation performed, and call out config or API contract changes. Link related issues or design docs when relevant.

Before committing or pushing AI-related changes, run `make check-ai`. If it fails, fix the issue before committing; when reporting work, include the failed check and the fix applied.

## Configuration & Contributor Notes
Model metadata is defined in `src/loushang/ai/model/models.json`; keep examples and docs aligned with it. Treat `examples/ai/` as public guidance: main examples should show the shortest supported path, while protocol-heavy or custom-registry flows should be marked advanced.

---
> Source: [zhnt/loushang](https://github.com/zhnt/loushang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
