## intake-skill

> - `src/intake_skill/` contains the Python package and CLI implementation.

# AGENTS.md - Intake Skill

## Layout

- `src/intake_skill/` contains the Python package and CLI implementation.
- `tests/` contains offline pytest coverage. Tests must not read real Voice Memos data.
- `docs/` contains PRD, RFC, test notes, and working log.
- `skills/skill_intake.md` is the public AI-agent skill entrypoint.
- `prompts/` contains prompt templates used by installer agents and Codex postprocessing.

## Operating Rules

1. Keep the project Voice Memos only. Do not add microphone recording, speaker recognition, diarization, reference voices, or private life-record imports.
2. Documentation in this repo stays English and AI-installer oriented.
3. Keep mock engines offline and deterministic so `uv pip install -e '.[dev]'` and `python -m pytest -q` work without network or private data.
4. After substantive changes, update `docs/working.md` under Changelog and add concrete Lessons Learned when relevant.
5. Treat `data/`, `logs/`, `.env`, and generated audio/report artifacts as local operator state, not source files.

## Environment

Python 3.10+. Use `.venv` and `uv pip install -e '.[dev]'` from the repository root.

---
> Source: [grapeot/intake-skill](https://github.com/grapeot/intake-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
