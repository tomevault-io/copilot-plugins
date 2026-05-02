## sinfim

> This repository is the Sinfim.uz workspace.

# AGENT INSTRUCTIONS

## Project

This repository is the Sinfim.uz workspace.

Sinfim.uz is a multi-tenant online school platform for schools, education centers, teachers, mentors, and course-selling teams in Uzbekistan.

## Workspace Map

| Path | Purpose |
|------|---------|
| `docs/` | Product docs, UX/tech decisions, UI design output, shared AI context |
| `backend-go/` | Go backend application, based on the Go enterprise blueprint |
| `frontend-vue/` | Vue 3 + TypeScript frontend application, based on the Vue web blueprint |

## Read First

For any task, read:

- `docs/ai-context/HANDOFF.md`
- `docs/ai-context/SESSION.md`
- `docs/product/brand-constants.md`
- `docs/product/tech-doc.md`
- `docs/product/ux-doc.md`

For UI/frontend work, also read:

- `docs/product/ui-design-review.md`
- `docs/design/sinfim-design/README.md`

## Subproject Rules

Backend work:

- Work inside `backend-go/`.
- Follow `backend-go/AGENTS.md` and `backend-go/CLAUDE.md`.
- Use the backend-specific `.claude/` and `.codex/` agents in `backend-go/`.

Frontend work:

- Work inside `frontend-vue/`.
- Follow `frontend-vue/AGENTS.md` and `frontend-vue/CLAUDE.md`.
- Use the frontend-specific `.claude/` and `.codex/` agents in `frontend-vue/`.

Docs/product planning work:

- Work under `docs/`.
- Keep `docs/ai-context/` current when decisions change.

## Canonical Decisions

- Platform name: `Sinfim.uz`.
- Domain: `sinfim.uz`.
- Organization URL model: `sinfim.uz/{school-slug}`.
- MVP auth: phone number + password/temporary access code.
- No SMS OTP in MVP.
- No Telegram login in MVP.
- MVP video strategy: Telegram stream reference.

## Verification

Before finishing implementation work, run the relevant project verification:

- Backend: check `backend-go/Makefile`; expected final commands are `make lint`, `make test`, `make test-system`.
- Frontend: check `frontend-vue/package.json`; expected final command is `npm run build`.

Do not invent project conventions when `docs/`, `backend-go/`, or `frontend-vue/` already define them.

---
> Source: [asadbek21coder/sinfim](https://github.com/asadbek21coder/sinfim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
