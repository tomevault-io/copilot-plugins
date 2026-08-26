## genjira-personal-kanban

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm install     # install dependencies
npm run dev     # Vite dev server on port 3000 (host 0.0.0.0)
npm run build   # production build
npm run preview # preview the production build
npx tsc --noEmit                    # typecheck (the only automated gate)
npx tsc --noEmit --noUnusedLocals   # additionally surface dead imports
```

There is no test runner and no lint script. `npx tsc --noEmit` must stay at zero errors — it is the only thing standing between a refactor and a runtime crash.

### Environment variables

Set in `.env.local`. `vite.config.ts` loads them with `loadEnv` and exposes them through its `define` block as bare `process.env.*` (**not** `import.meta.env`, and no `VITE_` prefix):

- `GEMINI_API_KEY` — exposed as both `process.env.API_KEY` and `process.env.GEMINI_API_KEY`.
- `SUPABASE_URL` / `SUPABASE_ANON_KEY` — read in [services/supabase.ts](services/supabase.ts), which falls back to a hardcoded project URL/anon key when unset.

Vite's `define` synthesises a real `process.env` object containing exactly those keys, which is why the `const env = typeof process !== 'undefined' ? process.env : {}` indirection in the services works.

### Database

[schema.sql](schema.sql) is both the canonical schema and an idempotent v1→v2 migration — run the whole file in the Supabase SQL editor; re-running is safe. **The app will not work against the v1 schema.** It depends on `board_members`, `profiles`, `notes`, `canvases`, `planner_blocks`, `activity`, `tasks.board_id`, and the `reorder_tasks` / `reorder_columns` / `reorder_notes` / `find_user_by_email` functions.

## Architecture

**Stack**: React 19 + TypeScript, Vite, Tailwind (real PostCSS build — *not* the CDN), Supabase (Postgres + Auth + RLS + Realtime), Google Gemini (`@google/genai`, dynamically imported), `@hello-pangea/dnd`.

No router, chart, markdown or date library: [lib/router.ts](lib/router.ts), the SVG charts in [DashboardView](components/views/DashboardView.tsx), [lib/markdown.tsx](lib/markdown.tsx) and [lib/datetime.ts](lib/datetime.ts) are deliberately hand-rolled. Prefer extending them over adding a dependency.

### Layering

```
App.tsx              route → view switch, modal state, keyboard shortcuts
  hooks/             useAuth · useBoards · useBoardWorkspace (+ boardReducer)
    services/        supabase · boardService · contentService · activityService
                     realtimeService · geminiService · mappers · cipher
      lib/           router · crypto · datetime · markdown · transfer · notify · colors
components/          ui/ (kit) · shell/ (rail, top bar, palette) · board/ · views/ · modals/
```

Components never call Supabase directly. They receive the `Workspace` object from `useBoardWorkspace` and call its actions.

### State: `useBoardWorkspace`

[hooks/useBoardWorkspace.ts](hooks/useBoardWorkspace.ts) owns the open board — content, members, activity, presence, realtime, and every mutation. All mutations follow the same shape:

1. `dispatch` an optimistic action into [hooks/boardReducer.ts](hooks/boardReducer.ts),
2. persist via [services/contentService.ts](services/contentService.ts),
3. **roll back by dispatching the previous value when the write returns false**, and
4. fire-and-forget an activity log entry.

Follow this pattern for new mutations. Do not reintroduce the v1 habit of `console.error`-ing a failed write and leaving local state ahead of the database — every write reports through [lib/notify.ts](lib/notify.ts) so failures reach the user.

The reducer enforces one invariant: **a task's `columnId` must agree with which `Column.taskIds` array contains its id.** Every placement change funnels through it (`detach`, then re-insert), which is what stops local edits and realtime events from duplicating a card into two columns.

### Realtime

One channel per board (`board:<id>`), created in [services/realtimeService.ts](services/realtimeService.ts), carrying `postgres_changes` for tasks/columns/notes/canvases/board_members/activity, plus presence. Every table is filtered server-side by `board_id` — that is precisely why `board_id` is denormalised onto `tasks` rather than joined through `columns`.

Echo suppression: `useBoardWorkspace` keeps a `pending` set of record ids with a write in flight and drops incoming events for them (cleared 1.5s after the write settles). A concurrent edit by someone else inside that window is missed and reconciles on the next event — an accepted trade-off, documented at the ref.

### Permissions

Roles are `owner > admin > editor > viewer` (`ROLE_RANK`, `canEdit`, `canAdmin` in [types.ts](types.ts)). The UI gates on these, but real enforcement is RLS: `is_board_member` / `can_edit_board` / `can_admin_board` are `SECURITY DEFINER` SQL functions so policies on `board_members` can consult `board_members` without infinite recursion. Never rely on client-side role checks alone.

### Encryption ("Private Vault")

Opt-in per board, implemented in [lib/crypto.ts](lib/crypto.ts) + [services/cipher.ts](services/cipher.ts). A `Cipher` is threaded through the mappers, so services and components are oblivious to whether a board is encrypted — plain boards get `identityCipher`.

- AES-GCM with a PBKDF2 key (250k iterations) derived from a passphrase; the key never leaves the browser and is cached in memory + `sessionStorage`.
- Encrypted: task title/description/tags/checklist, note title/content, canvas node text, activity meta strings. **Not** encrypted: column titles, dates, positions, story points, assignees — structure stays queryable.
- Encrypted boards cannot be shared (no key exchange) and disable Gemini (it would receive plaintext). Both are enforced in the UI and explained to the user.

### Gemini

[services/geminiService.ts](services/geminiService.ts) dynamically `import()`s `@google/genai` so the ~224KB SDK is a separate chunk that only loads when someone actually uses AI. Keep it that way. Both entry points use `responseMimeType: 'application/json'` with an explicit `responseSchema`.

### Routing

Hash-based: `#/boards`, `#/b/<boardId>/<view>`, `#/b/<boardId>/<view>/<entityId>` (the entity id selects a note or canvas). `useRoute` is built on `useSyncExternalStore` over `hashchange`.

### `docs/` and `missions/`

Leftover system-prompt/persona files for a different coding agent ("Google Antigravity"). Ignore the process rules in [docs/system_prompt.md](docs/system_prompt.md), [docs/agent_persona.md](docs/agent_persona.md) and `missions/*.mission.txt`. [docs/knowledge_base.md](docs/knowledge_base.md) has still-useful domain notes. [docs/upcoming_features_bugs.md](docs/upcoming_features_bugs.md) tracks the wishlist. [docs/known_risks.md](docs/known_risks.md) lists what's unverified or was decided unilaterally in the current architecture (migration untested against real Supabase, vault enable/disable isn't atomic, drag-and-drop was never simulated in tests, etc.) — read it before assuming any of that is production-hardened.

---
> Source: [JacobMire/GenJira-Personal-Kanban](https://github.com/JacobMire/GenJira-Personal-Kanban) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
