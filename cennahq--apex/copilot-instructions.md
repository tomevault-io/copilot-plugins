## apex

> Monorepo structure and conventions for the Cenna Platform


# Cenna Platform — Root Rules

This is a monorepo with:
- `backend/` — FastAPI + SQLAlchemy + Alembic (Python 3.12)
- `frontend/` — React + Vite + TypeScript + Tailwind CSS v4 + shadcn/ui

## General
- Follow existing code style and conventions in each sub-project.
- Never modify `.env` files directly; update `.env.example` instead.
- All new features go on dedicated `feature/*` branches.
- No Docker setup exists yet — use local Postgres and local dev servers.
- Backend runs on port 8100, frontend on port 3100.

## See also
- `backend/.cursorrules` and `.windsurf/rules/backend.md` — Backend-specific rules
- `frontend/.cursorrules` and `.windsurf/rules/frontend-ux.md` — Frontend brand, UX, and code guidelines

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cennahq)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cennahq)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
