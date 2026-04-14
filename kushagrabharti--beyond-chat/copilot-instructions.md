## beyond-chat

> This file defines the operating contract for **all coding agents** working in this repository.

# AGENTS.md

This file defines the operating contract for **all coding agents** working in this repository.

If you are an automated coding agent, read this file fully before making any change.

## 1) Mission Context

Beyond Chat is a modular AI workspace. The product is organized into dedicated studios (writing, research, image, data, finance) and artifact-centric workflows rather than long chat threads.

Your role as an agent is to make safe, minimal, correct changes that move the repository toward production readiness.

## 2) Hard Rules (Non-Negotiable)

### Package/runtime policy

- Frontend must use **npm**.
- Backend must use **uv only**.
- Never use `bun`, `yarn`, or `pnpm` for frontend dependency management.
- Never use `pip` for backend dependency management.

### Scope discipline

- Do only what the current task asks.
- Do not refactor unrelated code.
- Do not add extra features not requested.
- Do not rename files/symbols unless required for the task.

### Quality discipline

- Prefer root-cause fixes over surface patches.
- Keep implementations simple and explicit.
- Preserve existing code style and conventions.
- Update docs when behavior/setup changes.

## 3) Mandatory Startup Checklist (Run Before Work)

Every agent session must begin with this sequence:

1. Read `README.md` and this file.
2. Import/load **ALL available agent skills** before planning or coding.
3. Inspect repository state and active files related to the request.
4. Confirm toolchain compliance (npm frontend, uv backend) for any command you run.

### Skill-loading requirement

Agents must actively load all discoverable skills from the local agent skill registry so they operate with the latest instructions, templates, and safeguards.

- If a skill system supports commands, run the equivalent of `import all skills` / `load all skills` at session start.
- If auto-discovery is unavailable, manually read each available skill instruction source before coding.
- If no skills are available, continue and note that skill import was not possible.

## 4) Repository Map

- `frontend/` → production frontend (React + TypeScript + Vite, managed with npm)
- `frontend-mock/` → visual mock variants / design sandbox
- `backend/` → FastAPI backend (managed with uv)
- `backend/sql-related-files/` → SQL and schema-related assets

## 5) Current Stack and Tools

- Frontend: React, TypeScript, Vite
- Backend: FastAPI, Uvicorn
- JS tooling: npm
- Python tooling: uv
- Planned platform integrations: Supabase, OpenRouter, Vercel

## 6) Standard Local Commands

Use only these toolchain-compliant commands.

### Backend

```powershell
cd backend
uv sync
uv run uvicorn src.main:app --reload --host 127.0.0.1 --port 8000
```

### Frontend

```powershell
cd frontend
npm install
npm run dev -- --host 127.0.0.1 --port 5173
npm run build
```

## 7) API/Connectivity Baseline

At minimum, preserve the local integration path:

- Backend health endpoint: `GET /api/health`
- Frontend dev server proxies `/api/*` to backend (`127.0.0.1:8000`)

After backend/frontend edits, validate this path still works.

## 8) Execution Protocol for Agents

When implementing a task:

1. Gather context (relevant files only).
2. Plan minimal edit set.
3. Apply focused changes.
4. Run targeted validation first (build/lint/endpoint checks near changed code).
5. Summarize exactly what changed and any follow-up actions.

## 9) Validation Requirements

After changes, run what is relevant:

- Frontend code changed → `npm run build` in `frontend/`
- Backend code changed → start/verify backend endpoint(s) using `uv run ...`
- Cross-stack changes → verify `/api/health` through frontend and backend URLs

Do not attempt to fix unrelated failing tests/issues unless explicitly asked.

## 10) Security and Data Handling

- Do not hardcode secrets or tokens.
- Use environment variables for credentials/config.
- Do not expose private data in logs or docs.
- Keep dependencies minimal and justified.

## 11) Commit/PR Hygiene (If Asked)

- Keep commits small and scoped.
- Use clear messages describing behavior changes.
- Include README/doc updates for setup/workflow changes.

## 12) Definition of Done

A task is done when:

- Requested behavior is implemented correctly.
- Toolchain policy (npm/uv) is fully respected.
- Relevant validation checks pass.
- Documentation is updated when needed.
- No unrelated code was modified.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KushagraBharti) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
