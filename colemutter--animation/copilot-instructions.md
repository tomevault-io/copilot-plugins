## animation

> Agent configs and related files live in `agents/`.

# AGENTS

Agent configs and related files live in `agents/`.

Project documentation lives in `docs/`.

## Project stack

- **Frontend:** React + Vite + TypeScript in `frontend/`. Dev server: `npm run dev` (port 5173).
- **Backend:** FastAPI (Python) in `backend/`, managed with uv. Dev server: `uv run fastapi dev main.py` (port 8000).
- The Vite dev server proxies `/api/*` to the backend (see `frontend/vite.config.ts`).

## Skill routing

When the user's request matches an available skill, route to that skill and follow its instructions before answering directly. When in doubt, use the matching skill.

Key routing rules:
- Product ideas/brainstorming -> invoke /office-hours
- Strategy/scope -> invoke /plan-ceo-review
- Architecture -> invoke /plan-eng-review
- Design system/plan review -> invoke /design-consultation or /plan-design-review
- Full review pipeline -> invoke /autoplan
- Bugs/errors -> invoke /investigate
- QA/testing site behavior -> invoke /qa or /qa-only
- Code review/diff check -> invoke /review
- Visual polish -> invoke /design-review
- Ship/deploy/PR -> invoke /ship or /land-and-deploy
- Save progress -> invoke /context-save
- Resume context -> invoke /context-restore
- Author a backlog-ready spec/issue -> invoke /spec

---
> Source: [colemutter/AnImation](https://github.com/colemutter/AnImation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
