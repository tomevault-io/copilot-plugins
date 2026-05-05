## yorumi

> This file defines how coding agents should work in this repo so execution stays consistent, fast, and context-aware.

# Yorumi Agent Playbook

This file defines how coding agents should work in this repo so execution stays consistent, fast, and context-aware.

## Mission
- Ship minimal, correct fixes with clear validation.
- Preserve existing behavior unless a change request explicitly says otherwise.
- Prefer root-cause fixes over local patches.

## Project Context
- Stack: React 19 + TypeScript + Vite (frontend), Express + TypeScript backend scraper API.
- Frontend source: `src/`
- Backend source: `backend/src/`
- Build outputs: `dist/`, `backend/dist/` (do not hand-edit)

## Default Workflow
1. Understand scope and impacted layer (`src`, `backend/src`).
2. Inspect existing patterns in nearby files before editing.
3. Make the smallest coherent change.
4. Run the most relevant validation commands.
5. Report exactly what changed, why, and how it was verified.

## Execution Rules (Do)
- Use `rg`/`rg --files` for fast search.
- Reuse existing utilities/hooks/services before creating new ones.
- Keep API contracts stable unless the task explicitly includes contract changes.
- Match local naming and file organization conventions.
- Prefer TypeScript-safe fixes over `any`/casts.
- When touching scraper logic, preserve fallback behavior and error handling.
- Keep frontend and backend changes logically separated in commits/summary.

## Guardrails (Do Not)
- Do not edit generated/build artifacts in `dist*` or `backend/dist`.
- Do not introduce broad refactors during bugfix tasks.
- Do not add new dependencies unless necessary and justified.
- Do not hardcode secrets or tokens; use environment variables.
- Do not silently change ports, routes, or IPC channel names.
- Do not weaken error handling in scraper/network flows.

## Validation Matrix
Run only what is relevant to changed areas, but always run at least one verification command.

- Frontend/UI changes:
  - `npm run lint`
  - `npx tsc -p tsconfig.app.json --noEmit`
  - `npm run build` (if change affects bundling/routes/config)
- Backend/API/scraper changes:
  - `npm run dev --prefix backend` (smoke start)
  - `npm run build --prefix backend`
If a command cannot run (time/tooling/env), state that explicitly and provide the next best verification evidence.

## Change Heuristics
- Bugfix request: reproduce signal -> identify failing path -> patch root cause -> verify with targeted command.
- Feature request: implement vertical slice first -> wire data flow -> polish UI -> validate build/lint.
- Refactor request: preserve behavior, include before/after risk notes, and keep diff scoped.

## File-Level Guidance
- `src/features`, `src/components`: keep presentational logic separate from data-fetching logic.
- `src/services`: external I/O and API wrappers only; keep pure transforms in `src/utils`.
- `src/hooks`: reusable stateful behavior; avoid embedding endpoint constants directly.
- `backend/src/api`: request/response shaping and orchestration.
- `backend/src/scraper`: source-specific scraping/parsing with resilient fallbacks and logging.

## Done Criteria
A task is done only when all are true:
- Requested behavior is implemented.
- No unrelated files were changed.
- Relevant checks were run (or inability was clearly reported).
- Summary includes changed files and validation commands used.

## Response Format for Agents
- Start with result status.
- List key file changes with paths.
- List validation commands run and outcomes.
- End with short residual risks or follow-up items (only if real).

---
> Source: [davenarchives/Yorumi](https://github.com/davenarchives/Yorumi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
