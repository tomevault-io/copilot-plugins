## calculator-app

> All production code belongs under `src/calculater/`. Organize modules by feature (e.g., `src/calculater/operations/addition.py` for arithmetic, `src/calculater/cli.py` for the command-line interface). Shared helpers go in `src/calculater/utils/`. Tests mirror the layout in `tests/`, such as `tests/operations/test_addition.py`. Store reusable fixtures under `tests/fixtures/`. Assets like diagrams or sample inputs belong in `assets/`.

# Repository Guidelines

## Project Structure & Module Organization
All production code belongs under `src/calculater/`. Organize modules by feature (e.g., `src/calculater/operations/addition.py` for arithmetic, `src/calculater/cli.py` for the command-line interface). Shared helpers go in `src/calculater/utils/`. Tests mirror the layout in `tests/`, such as `tests/operations/test_addition.py`. Store reusable fixtures under `tests/fixtures/`. Assets like diagrams or sample inputs belong in `assets/`.

## Build, Test, and Development Commands
Create a virtual environment with `python -m venv .venv` and activate it before working. Install dependencies via `python -m pip install -r requirements.txt`. Run the application locally with `python -m calculater.cli` (ensures imports resolve through the package). Execute the test suite using `python -m pytest`. For release validation, run `python -m build` to produce wheels and source archives.

## Coding Style & Naming Conventions
Follow PEP 8 with 4-space indentation, maximum line length 100. Use snake_case for functions and variables, PascalCase for classes, and descriptive module names (e.g., `operations_division.py`). Type hints are required for public functions; prefer dataclasses over bare tuples when returning structured data. Format code with `ruff format` and lint using `ruff check src tests`.

## Design Principles
Code should adhere to SOLID by keeping each class responsible for one behavior, injecting dependencies, and respecting interface contracts. Apply DRY by extracting shared helpers into `src/calculater/utils/` or pytest fixtures, avoiding copy-paste logic, and centralizing constants. Keep implementations KISS: favor straightforward control flow, prefer pure functions for calculations, and document any unavoidable complexity with a short comment.

## Testing Guidelines
All new features require unit tests in `tests/` that match the module path. Name test files `test_<feature>.py` and functions `test_<behavior>`. Prefer `pytest` fixtures for setup; keep external calls mocked. Maintain at least 90% coverage by running `python -m pytest --cov=src/calculater --cov-report=term-missing`. Add regression tests whenever a bug is fixed.

## Commit & Pull Request Guidelines
Use Conventional Commit prefixes (`feat:`, `fix:`, `docs:`, etc.) followed by a concise summary. Commits should remain focused; refactors and behavior changes belong in separate commits. Pull requests must include a clear problem statement, outline of the solution, and a checklist of tests executed. Link related issues and attach console output or screenshots demonstrating the change where relevant.

---
> Source: [1106601034/calculator-app](https://github.com/1106601034/calculator-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
