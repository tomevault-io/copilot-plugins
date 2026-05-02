## scale

> Agent guidance for this repository. Prefer this file for repo-specific operating context.

# AGENTS.md

Agent guidance for this repository. Prefer this file for repo-specific operating context.

## Project Summary

This is a small Python project for discovering and monitoring Acaia coffee scales over Bluetooth LE, primarily to log bird weights to CSV, with optional Gmail-based battery alerts.

Active scripts in the repo:

- `monitor.py`: main monitoring loop, auto-tare logic, CSV logging, battery checks, Gmail alerts
- `discover.py`: Bluetooth discovery for nearby Acaia devices
- `simulator.py`: simulated scale behavior for development without hardware
- `test_simulator.py`: lightweight simulator test runner

## Tooling

- Python: `>=3.12`
- Package manager / runner: `uv`
- Linting / formatting: `ruff`

Prefer running Python commands through `uv run ...`.

Common commands:

```bash
uv run python monitor.py --simulate
uv run python discover.py
uv run python test_simulator.py
uv run ruff check .
uv run ruff format .
```

## Working Conventions

- Treat checked-in source files as the source of truth if docs and metadata disagree.
- Avoid changing local output artifacts unless the task requires it.
- Assume Bluetooth hardware may not be present; prefer simulator-based verification when possible.
- Gmail OAuth credentials are optional local state; do not create, overwrite, or commit credential files unless explicitly asked.
- Keep edits minimal and targeted. This repo is small and mostly script-based.

## Local State And Outputs

The following paths may contain local data rather than source code:

- `bird_weights.csv`
- `dead/`
- `credentials.json`
- `~/.config/acaia-scale/credentials.json`
- `token.json`

These files are currently untracked. Do not delete, rewrite, or normalize them unless the user explicitly asks.

## Validation Strategy

Use the least expensive validation that matches the change:

- For pure logic changes: `uv run python test_simulator.py`
- For lint-sensitive edits: `uv run ruff check .`
- For formatting-sensitive edits: `uv run ruff format .`
- For hardware flows: prefer discovery or monitor commands only if the task specifically depends on real BLE access

## Known Repo Mismatch

`README.md` accurately describes the scale-monitoring workflow, but `pyproject.toml` also includes some dependencies and Ruff first-party module names that do not match the current file layout. Do not assume those missing modules exist.

## Agent Notes

- If a task mentions "the standard agent file", use this `AGENTS.md`.

---
> Source: [wwade/scale](https://github.com/wwade/scale) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
