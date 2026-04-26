## g-rard

> A self-hosted Jira + Confluence clone for managing home renovation projects (2 users).

# Gérard — Claude Code Context

## What is Gérard?

A self-hosted Jira + Confluence clone for managing home renovation projects (2 users).
Built as a personal monorepo, runs locally via Docker Compose.

---

## Language rule — CRITICAL

**All code must be in English. Never use French in source code.**

This applies to:
- Variable names, function names, class names, type names
- Comments and JSDoc
- Log messages and error strings
- Commit messages
- File names

**French is only allowed in one place: the translation file `client/src/i18n/locales/fr.json`.**

The app displays French to users via i18next — use translation keys (e.g. `t("task.new")`) rather than hardcoded French strings. Never write raw French in `.tsx`, `.ts`, or `.json` files outside of `fr.json`.

---

## Monorepo structure

```
gerard/
├── server/          Fastify + TypeScript + Prisma + MariaDB
├── client/          React + Vite + TypeScript + TanStack Query/Router + Tailwind + shadcn/ui
├── mcp/             MCP server for Claude Code integration
├── docker-compose.yml
└── PLAN.md          Full development plan with schema and API routes
```

npm workspaces — run commands from the root or inside `server/`, `client/`, `mcp/`.

---

## Tech stack

| Layer | Choice |
|---|---|
| Backend | Fastify + TypeScript |
| ORM | Prisma |
| Database | MariaDB |
| Frontend | React 18 + Vite + TypeScript |
| Server state | TanStack Query |
| Routing | TanStack Router (file-based, type-safe) |
| UI | shadcn/ui + Tailwind CSS |
| Drag & drop | dnd-kit |
| Rich text | Tiptap |
| Auth | Sessions (bcrypt + fastify-session) |
| i18n | i18next + react-i18next |
| Testing | Vitest |
| Deploy | Docker Compose |

---

## i18n setup

Translation files: `client/src/i18n/locales/en.json` and `fr.json`.
Config: `client/src/i18n/index.ts` — initialized as a side-effect in `main.tsx`.
Language switcher component: `client/src/components/ui/LanguageSwitcher.tsx`.
Persistence: localStorage key `gerard_lang`.
Fallback: `en`.

Namespace structure: `common`, `auth`, `dashboard`, `project`, `settings`, `kanban`, `task`, `epic`, `ticket`, `activity`, `wiki`, `search`, `attachments`, `statuses`, `priorities`, `linkTypes`, `sidebar`.

---

## Key shared constants (client)

- `client/src/lib/utils.ts` — `STATUS_LABELS`, `PRIORITY_LABELS`, `cn()`
- `client/src/api/taskLinks.ts` — `LINK_TYPE_LABELS`

---

## Database key enums (Prisma / MariaDB)

- Status: `a_faire`, `en_cours`, `termine`, `bloque`
- Priority: `basse`, `normale`, `haute`, `urgente`
- Link types: `blocks`, `is_blocked_by`, `relates_to`, `duplicates`, `is_duplicated_by`, `causes`, `is_caused_by`

These enum values are French-origin identifiers — they are database constants, not user-facing strings, and must not be changed.

---

## API

All routes under `/api/`. Auth via session cookie. MCP server uses `x-api-key` header (`GERARD_API_KEY` env var).

See `PLAN.md` for the full route list.

---

## Phases

- Phase 1 — MVP Kanban ✅
- Phase 2 — Core features (labels, comments, activity) ✅
- Phase 3 — Attachments, rich text, Wiki ✅
- Phase 4 — Search & export ✅
- Phase 5 — MCP server ✅
- Phase 6 — Ticket types, Epic hierarchy, task links ✅

---

## Dev conventions

- Prefer editing existing files over creating new ones.
- Do not add features or refactor beyond what is asked.
- Do not add comments unless the logic is non-obvious.
- Tests live alongside routes: `server/src/routes/*.test.ts` (Vitest + supertest).
- The app targets 2 users — do not over-engineer for scale.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/arsebac) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
