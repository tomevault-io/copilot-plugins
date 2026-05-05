## beads-dashboard

> This project uses **bd (beads)** for issue tracking. Run `bd prime` for workflow context, or install hooks (`bd hooks install`) for auto-injection.

# Repository Guidelines

## Issue Tracking
This project uses **bd (beads)** for issue tracking. Run `bd prime` for workflow context, or install hooks (`bd hooks install`) for auto-injection.  
**Quick reference:** `bd ready` (find work), `bd create "Title"` (new issue), `bd close <id>` (complete), `bd sync` (sync with git). Claim tasks via `bd update <id> --status in_progress` before coding and file/close items during handoff.

## Project Structure & Module Organization
Source lives in `src`: `src/client` contains the React UI, `src/server` holds the Express/Socket.IO backend, and `src/shared` hosts types reused by both via the configured aliases (`@/`, `@server/`, `@shared/`). CLI wiring is under `bin`, static assets in `public`, distribution builds in `dist`, and automated tests plus fixtures in `tests`.

## Build, Test, and Development Commands
- `npm run dev` – parallel Vite client and `tsx` server for live TypeScript edits.
- `npm run build` – chains `build:client` and `build:server` to refresh `dist/`.
- `npm run start -- /path/to/project` – run the compiled CLI/Express server against a `.beads` directory.
- `npm run preview` – static preview of the built client for UX checks.
- `npm run typecheck` – strict TypeScript across client/server projects.

## Coding Style & Naming Conventions
Code is modern TypeScript with strict compiler settings, React function components, and 2-space indentation (see `src/client/App.tsx`). Use PascalCase for components, `use*` for hooks, kebab-case for CLI files, and colocate helpers with their feature. Import shared contracts through the aliases instead of relative paths. Server modules should stay async/await friendly, avoid mutable globals, and remain ESM-compatible (`"type": "module"`).

## Testing Guidelines
Vitest powers the suite under `tests/unit`, with `tests/setup.ts` registering shared mocks and `tests/fixtures` storing canonical `.beads` samples. Name new specs `*.test.ts` and group them with similar coverage (e.g., router specs in `tests/unit/api.test.ts`). Run `npm test` for quick feedback, `npm run test:coverage` before release-level changes, and extend fixtures when new issue fields appear so metrics stay reproducible.

## Commit & Pull Request Guidelines
Follow the Conventional Commit pattern (`feat(scope):`, `fix:`, etc.) and reference the associated bead ID when possible, e.g., `feat(priority): show SLA breach (bd#123)`. Before opening a PR, ensure `npm run build`, `npm test`, and `npm run typecheck` succeed, then describe the change, attach screenshots for UI tweaks, and note how reviewers can reproduce with `npm run dev`. Keep commits focused, update docs when behavior changes, and close/comment on the linked bd issue once merged.

---
> Source: [rhydlewis/beads-dashboard](https://github.com/rhydlewis/beads-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
