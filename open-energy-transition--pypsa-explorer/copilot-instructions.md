## pypsa-explorer

> The dashboard code lives under `src/pypsa_explorer`, with `app.py` orchestrating app creation, `cli.py` exposing the entry point, and feature-specific logic split into `callbacks/`, `layouts/`, and `utils/`. Tests in `tests/` mirror the package layout. Keep documentation in `docs/`, notebooks in `examples/`, and transient `.nc` files in `uploaded_networks/` (never commit them).

# Repository Guidelines

## Project Structure & Module Organization
The dashboard code lives under `src/pypsa_explorer`, with `app.py` orchestrating app creation, `cli.py` exposing the entry point, and feature-specific logic split into `callbacks/`, `layouts/`, and `utils/`. Tests in `tests/` mirror the package layout. Keep documentation in `docs/`, notebooks in `examples/`, and transient `.nc` files in `uploaded_networks/` (never commit them).

## Build, Test, and Development Commands
Bootstrap with `pip install -e ".[dev]"` and `pre-commit install`. `make run` launches the Dash app against the bundled demo network. Running `pypsa-explorer` with no arguments opens the landing page, where contributors can drag-and-drop `.nc` files or load the bundled example via the UI. `make test` executes the pytest suite, while `make test-cov` emits HTML and XML coverage reports under `htmlcov/` and `coverage.xml`. `make lint` chains Ruff and mypy; `make format` applies Black and Ruff auto-fixes. Build docs with `make docs` inside the repo root.

## Coding Style & Naming Conventions
We target Python 3.12, four-space indentation, and 125-character lines. Install the repo’s pre-commit hooks so Black, Ruff, and mypy run on staged files automatically; use `make format` for a full-tree cleanup before large refactors. Favor DRY patterns: extract shared callbacks/components into `layouts/` or `utils/` to keep boilerplate low. Ruff also enforces import order and bugbear checks as configured in `pyproject.toml`. Keep modules and variables snake_case, classes PascalCase, and Dash component IDs descriptive (e.g. `network_map_graph`). Type annotations are required; prefer explicit `TypedDict` or dataclasses over loose dictionaries. House configuration constants and themes in `config.py`.

## Testing Guidelines
Pytest discovers files named `test_*.py`, classes prefixed with `Test`, and functions named `test_*`. Reuse fixtures from `tests/conftest.py` and keep integration coverage for callbacks in `tests/test_callbacks_*.py`. Maintain coverage by running `pytest --cov=pypsa_explorer`; resolve any `term-missing` warnings before merge. Add regression tests when introducing new callbacks, CLI flags, or layout components.

## Commit & Pull Request Guidelines
Recent history favors Conventional Commit prefixes (`feat:`, `fix:`, `refactor:`); keep summaries under 72 characters and focused on one change. Squash noisy fixups locally. Pull requests should outline motivation, highlight affected views, and note manual or automated tests run. Link related issues and attach screenshots or GIFs for UI updates. Ensure `make lint` and `make test` pass before requesting review.

## Data & Configuration Notes
Large or proprietary `.nc` files stay outside source control; keep temporary assets in `uploaded_networks/`. When adjusting shared defaults in `config.py`, update the documentation and provide example usage in `examples/` when relevant.

---
> Source: [open-energy-transition/PyPSA-Explorer](https://github.com/open-energy-transition/PyPSA-Explorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
