## engineering

> Always-on engineering rules for RivalLens. Use before any code edit, refactor, Git operation, or env/config change.


# RivalLens Engineering Rules

## Repository Model

Single-repository monorepo. Top-level packages:

- `backend/` — Python 3.11 + FastAPI + LangGraph agent core, PostgreSQL persistence.
- `frontend/` — React + Vite + TypeScript visualization console.
- `docs/` — source-of-truth specs (problem, demo, architecture, schema, ownership, prior art).
- `backend/skills/` — Skill Library (`applies_to` in `qa_rule`, `prompt_template`, `source_routing`; loaded via `load_skill` / `read_skill_file`).
- `backend/demo_fixtures/` — demo seeds (competitor autocomplete source, not a hard constraint set).
- `configs/`, `data/`, `scripts/` — supporting material.

## Git Safety

- Never use `git add .` or `git add -A` from the repo root.
- Never run destructive commands (`git reset --hard`, `git clean -fd`, recursive forced deletes) without explicit user approval.
- Preserve unrelated dirty files when staging.
- Prefer explicit file paths for staging and edits.

## Env and Secret Safety

- Never commit real `.env` values, tokens, API keys, credentials, cookies, or private URLs.
- Edit `.env.example` for placeholders; copy to `.env` locally and keep it gitignored.
- Do not log full JWTs, API keys, or `Authorization` headers.

## Engineering Discipline

- Read existing patterns before adding new ones.
- Fail fast at boundaries — do not silently return empty defaults to hide failures.
- No bare `except` / `except Exception` — catch specific exceptions.
- Type hints on every Python function signature.
- Comments explain WHY, never WHAT. No narrative comments that restate code.
- No wrapper classes that merely delegate (avoid Ghost Layers).
- YAGNI: do not add features beyond what is explicitly requested.

## Temporary Integration

Keep temporary or experimental integrations visibly temporary. Avoid persistence until explicitly requested.

---
> Source: [wanghong5233/RivalLens](https://github.com/wanghong5233/RivalLens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
