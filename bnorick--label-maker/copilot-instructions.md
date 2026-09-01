## label-maker

> Application code lives in `src/label_maker/`. Core rendering and configuration are in `label.py` and `label_config.py`; printer integration is in `printer.py`; the Litestar application is in `api.py`; and CLI commands live under `cli/commands/`. HTML/Jinja templates are packaged in `src/label_maker/templates/`, while reusable label definitions such as `templates/lego.toml` live at the repository root. Tests are in `tests/`. Files named `test_unit*.py` exercise logic directly; numbered `test_*.py` files define rendering scenarios. `vendor/brother_ql/` is the editable workspace dependency. Treat `annex/` as reference or generated material, not production source.

# Repository Guidelines

## Project Structure & Module Organization

Application code lives in `src/label_maker/`. Core rendering and configuration are in `label.py` and `label_config.py`; printer integration is in `printer.py`; the Litestar application is in `api.py`; and CLI commands live under `cli/commands/`. HTML/Jinja templates are packaged in `src/label_maker/templates/`, while reusable label definitions such as `templates/lego.toml` live at the repository root. Tests are in `tests/`. Files named `test_unit*.py` exercise logic directly; numbered `test_*.py` files define rendering scenarios. `vendor/brother_ql/` is the editable workspace dependency. Treat `annex/` as reference or generated material, not production source.

## Build, Test, and Development Commands

### Use `./tasks` as the canonical command interface

Run `./tasks` to list the supported project commands and their usage. Use these recipes instead of invoking deptry, Ruff, ty, or pytest directly through `uv run`; this keeps local development and automated verification on the same paths.

When an agent needs to run `./tasks check` or a narrower `./tasks` verification command and the sandbox blocks execution because `uv` needs writable cache access outside the workspace, ask the user for permission to run the relevant command instead of treating that sandbox failure as a project failure.

Quality and test commands:

- `./tasks check`: run the complete non-mutating quality gate: deptry, Ruff lint, Ruff format checking, ty, and pytest. Use this before handing off a change. Any failed step fails the recipe.
- `./tasks fix`: apply Ruff's safe lint fixes and then format the source and tests.
- `./tasks deptry [ARGS...]`: check dependency declarations, forwarding optional deptry arguments.
- `./tasks ruff [check|fix] [ARGS...]`: check lint by default, or apply safe lint fixes with `fix`.
- `./tasks format [write|check] [ARGS...]`: format by default, or verify formatting without changes with `check`.
- `./tasks ty [ARGS...]`: type-check the application and tests, forwarding optional ty arguments.
- `./tasks test [PYTEST_ARGS...]`: run pytest; pass a path, `-k` expression, or other pytest options for focused runs.

If verification requires a command or option that `./tasks` does not expose, add or extend a recipe in `tasks` instead of leaving a direct tool invocation as the project workflow. Add an adjacent comment explaining when to use the new command or option so it appears clearly in `./tasks` output. Keep `./tasks check` non-mutating and ensure every verification failure propagates a nonzero exit status.

Environment, build, and application commands:

- `./tasks uv sync --dev`: create or update the environment from `uv.lock`, including development tools.
- `./tasks uv build`: build source and wheel distributions.
- `./tasks uv ARGS...`: run other uv environment, lockfile, or build-management commands.
- `./tasks run COMMAND [ARGS...]`: run a one-off command in the project environment. Do not use this as a permanent substitute for a named verification recipe.
- `./tasks clean`: remove `.venv`; recreate it with `./tasks uv sync --dev`.
- `./tasks start [ARGS...]`: serve the Litestar app on port 8800 and forward additional server arguments.
- `./tasks print TEXT...`: render and print labels using the default Ubuntu Mono configuration; this requires configured Brother QL hardware.

## Coding Style & Naming Conventions

Use four-space indentation and standard Python conventions: `snake_case` for modules, functions, and variables; `PascalCase` for classes and enums; and `UPPER_CASE` for constants. Add type annotations to public APIs and keep imports explicit. Ruff configuration is in `pyproject.toml`; run `./tasks check` before submitting changes. Keep template behavior in TOML/Jinja files where practical rather than embedding presentation details in command handlers.

## Testing Guidelines

Use pytest and name files `test_*.py`, classes `Test...`, and methods `test_...`. Add focused unit tests for parsing and configuration changes. Rendering features should also receive a numbered scenario based on `tests.LabelTestCase`. Tests must be deterministic and should not require a physical printer; isolate or mock USB/printer calls.

## Commit & Pull Request Guidelines

Recent history uses short, imperative summaries such as `Adds single label view`; avoid non-descriptive `WIP` commits in review-ready branches. Keep each commit scoped to one change. Pull requests should explain behavior and motivation, list verification commands, link relevant issues, and include screenshots or generated label samples for visual changes. Call out hardware-specific testing and any changes to `uv.lock` or the vendored printer package.

---
> Source: [bnorick/label-maker](https://github.com/bnorick/label-maker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
