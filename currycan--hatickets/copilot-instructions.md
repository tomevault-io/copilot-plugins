## hatickets

> `mobile/` is the primary automation path and contains the uiautomator2 (u2) bot, prompt parsing, config loading, and helper scripts. `shared/` holds Python utilities reused by the mobile module. `desktop/` is a legacy Tauri app (`desktop/src` for Vue, `desktop/src-tauri` for Rust); keep changes there narrowly scoped unless the task explicitly targets it. Tests live in `tests/unit/`, `tests/integration/`, and `tests/conftest.py`. Reference docs and design notes live in `docs/`.

# Repository Guidelines

## Project Structure & Module Organization

`mobile/` is the primary automation path and contains the uiautomator2 (u2) bot, prompt parsing, config loading, and helper scripts. `shared/` holds Python utilities reused by the mobile module. `desktop/` is a legacy Tauri app (`desktop/src` for Vue, `desktop/src-tauri` for Rust); keep changes there narrowly scoped unless the task explicitly targets it. Tests live in `tests/unit/`, `tests/integration/`, and `tests/conftest.py`. Reference docs and design notes live in `docs/`.

## Build, Test, and Development Commands

Run Python work from the repo root:

```bash
poetry install
poetry run pytest
poetry run pytest -m unit
poetry run pytest tests/unit/test_mobile_config.py
```

For mobile runs, connect an Android device via USB with debugging enabled, then execute `mobile/scripts/start_ticket_grabbing.sh --yes`. Desktop maintenance uses `cd desktop && yarn install && yarn tauri dev`; build with `yarn tauri build`.

## Coding Style & Naming Conventions

Follow the existing style in each module rather than introducing a new one. Python uses 4-space indentation, `snake_case` functions, `PascalCase` classes, and small focused helpers. Vue components use `PascalCase` filenames such as `Form.vue`; JS modules use `camelCase` exports and currently favor double quotes plus semicolons. Rust code uses `snake_case` functions and standard `cargo fmt` layout even though no repo-wide formatter is checked in.

## Testing Guidelines

Pytest is the primary test runner. Name files `test_*.py`, classes `Test*`, and functions `test_*`. Use markers already defined in `pyproject.toml`: `unit`, `integration`, and `slow`. The local Python test configuration enforces branch coverage and currently fails below 84%, so keep new work covered and add focused regression tests near the affected module.

## Commit & Pull Request Guidelines

Recent history favors short, imperative subjects like `Fix mobile CI tests and coverage gaps` and scoped prefixes when useful, for example `feat:` or `docs:`. Keep commits focused and explain behavior changes, not just files touched. PRs should summarize the user-visible impact, list verification commands you ran, link related issues or plans, and include screenshots only when desktop UI behavior changes.

## Security & Configuration Tips

Do not commit real credentials, device IDs, cookies, or local runtime configs. Start from `mobile/config.example.jsonc` and keep personal overrides in ignored local files such as `mobile/config.local.jsonc`.

---
> Source: [currycan/HaTickets](https://github.com/currycan/HaTickets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
