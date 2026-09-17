## mynotion

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm install
npm run dev       # Vite dev server at http://localhost:5173
npm run build     # tsc -b && vite build — this is also the type-check; run it before pushing
npm run preview   # serve the production build locally
```

There is no test runner or linter configured; `npm run build` (strict TypeScript) is the correctness gate. The `verify` skill in `.claude/skills/verify/` documents how to drive the built app in a headless browser end-to-end.

## Deployment

`.github/workflows/deploy.yml` is the only workflow: it builds and deploys to GitHub Pages on every push to `main` (or manual dispatch). The site is served under `/MyNotion/`, which CI passes via the `DEPLOY_BASE` env var picked up in `vite.config.ts` — local builds keep base `/`.

## Architecture

Client-side React 18 + TypeScript + Vite app. No router, no state library, no CSS framework. By default all data lives in `localStorage` and there is no server; the only other runtime dependencies are the **optional, env-gated** auth/cloud layer (`@clerk/clerk-react`, `@supabase/supabase-js` — see below). The app is an installable PWA (`public/manifest.webmanifest`, `public/sw.js`, registered in `src/main.tsx` on production builds).

### State

One global `AppState` (`src/types.ts`) holds all domain data: semesters, courses, tasks, pages, grades, flashcards, study sessions. Everything except semesters is scoped to a semester via `semesterId`; tasks/pages/grades/flashcards/sessions may also reference a `courseId`.

- `src/store/StoreContext.tsx` — `useReducer` provider; exposes `useAppState()` / `useDispatch()`. Every dispatch persists the whole state to localStorage.
- `src/store/reducer.ts` — routes actions to per-domain slice reducers in `src/store/slices/`. New state → new slice file + action union member here.
- `src/store/persistence.ts` — load/save plus **versioned schema migrations**. If you change what gets persisted (anything in `types.ts` that lands in `AppState`), bump `SCHEMA_VERSION` and add a step to `migrations`; this upgrades both existing localStorage data and imported backup files (`src/features/layout/backup.ts` reuses `migrateStoredState`/`serializeState`). Never change stored shapes without a migration — real users' data lives only in their browser.
- Import from the `src/store` barrel (`import { useAppState } from "../../store"`), not from individual store files.

### Optional cloud login & sync

`src/config.ts` reads `VITE_CLERK_PUBLISHABLE_KEY` / `VITE_SUPABASE_URL` / `VITE_SUPABASE_ANON_KEY` (see `.env.example`). Without them the app runs local-only exactly as before. With a Clerk key, `src/main.tsx` mounts `ClerkProvider` and `src/App.tsx` gates the workspace behind sign-in (`src/features/auth/`); each user gets a namespaced localStorage key via `StoreProvider`'s `userId` prop. With Supabase keys too, `src/features/cloud/CloudSync.tsx` mirrors state to the `app_state` table (schema + RLS in `supabase/schema.sql`; setup walkthrough in `docs/CLOUD_SETUP.md`). The deploy workflow passes these as optional repo variables/secrets.

### UI

- `src/contexts/` — cross-cutting UI state (theme, navigation, Quick Find, focus timer, undo toast), separate from domain state. Navigation is a simple `{ view, courseId, pageId }` object in `NavigationContext`, not a router; `src/ActiveView.tsx` switches on it, the last view is persisted to localStorage, and browser back/forward walk the view history via pushState. Destructive dispatches should go through `useUndoableDispatch` (`UndoContext`), which snapshots state and offers an Undo toast.
- `src/features/<name>/` — one folder per feature owning its views, components and logic (layout, semesters, courses, tasks, notes, editor, grades, timetable, dashboard, quick-find, calendar, flashcards, focus, auth, cloud). `src/components/` holds only small shared primitives (Modal, Field…).
- **Adding a feature**: folder under `src/features/`, slice under `src/store/slices/` if it needs state, wire the view into `src/ActiveView.tsx` and navigation into the sidebar/mobile tab bar in `src/features/layout/`.

### Conventions

- All styling lives in `src/styles.css`. It opens with a **design-token block** (spacing `--sp-*`, radii `--r-*`, type `--fs-*`, elevation `--shadow-*`, control heights `--control-h*`, state colours `--hover`/`--hover-strong`/`--overlay`); every rule below it composes those tokens. A raw px value or raw colour in a feature rule is how the UI drifted apart before — add a scale step rather than a one-off. Dynamic values (course colors, timetable positions, swipe offsets) are passed as CSS custom properties via helpers in `src/utils/cssVars.ts` — never inline styles. The app is responsive: mobile layout (bottom tab bar, drawer sidebar, bottom-sheet dialogs) is driven by CSS plus `useMediaQuery`.
- Shared UI shells rather than per-view markup: `PageHeader` opens every view, `SectionTitle` heads every group inside one, `.empty` is the only empty state (it drops its dashed border automatically inside a `.card`), and buttons/selects/inputs all share `--control-h` so any toolbar lines up. Reuse these instead of hand-rolling a header row or a muted "nothing here" line.
- Business logic belongs in pure functions (`src/utils/`, or per-feature modules like `taskFilters.ts`, `search.ts`) so it's testable in isolation; components read state through context hooks rather than prop drilling.
- Grades use the Swiss 1–6 scale (4.0 = pass); conversion and weighted-average rules live in `src/utils/grades.ts`.
- Dates are stored as ISO strings; helpers in `src/utils/date.ts`.

## Repository state

`main` is the single long-lived branch and the deploy branch: every push to it builds and deploys to GitHub Pages. The repo previously carried two divergent lines (an old `main` and a `claude/...` GitHub default); they were consolidated — main's extra features were ported onto the current architecture and the histories merged — and the working branches deleted. Do feature work on short-lived branches PR'd into `main`, and make sure `main` is also the GitHub default branch.

---
> Source: [SwissMiles/MyNotion](https://github.com/SwissMiles/MyNotion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
