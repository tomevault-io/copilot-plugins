## ai-investing-app

> - The live application is the Reflex app under `frontend/`.

# AI-investing-app Codex Notes

## Project Shape

- The live application is the Reflex app under `frontend/`.
- Treat `legacy_streamlit/` as historical unless current evidence says otherwise.
- Read `progress.md` before non-trivial work because it records repo-local handoff state.
- Do not touch unrelated dirty files. This repo often has active user changes.

## Implementation Defaults

- Prefer small, direct fixes that match the existing Reflex/Python structure.
- Keep docs aligned with code when changing runtime behavior, setup, or validation.
- Keep cross-project agent behavior in Codex memory or user-level skills, not in this repo.
- Keep project-specific durable decisions in `progress.md` after substantive work.

## Validation

Use the repo-local `.venv` tools when available. The proven validation stack is:

- `.venv\Scripts\python.exe -m compileall src frontend tests`
- `.venv\Scripts\ruff.exe check .`
- `.venv\Scripts\ruff.exe format --check .`
- `.venv\Scripts\python.exe -m pytest -q`
- `.venv\Scripts\reflex.exe export --frontend-only --no-zip` for Reflex/startup/UI changes

If one validation path is blocked by the local Windows/Codex environment, find a smaller real
smoke test and report the exact blocker.

Run these checks from a Codex thread rooted at this repository. If the repository is outside
the active writable roots, pytest temp files, `__pycache__`, `.states`, and Reflex `.web`
build output can fail even when the application is healthy. `python scripts/check.py` runs
the complete release stack without installing packages or rewriting source files.

---
> Source: [shusuke-appdev/AI-investing-app](https://github.com/shusuke-appdev/AI-investing-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
