## seasons-study-app

> Interactive study guide PWA for my daughter Rose, with an embedded basketball

# seasons-study-app

Interactive study guide PWA for my daughter Rose, with an embedded basketball
tournament hub and three separately-built sub-apps.

## Commands

```bash
npm install
npm run dev       # Vite dev server (default port 5173)
npm run build     # Builds main app into dist/
npm run preview   # Preview production build
npm run lint
```

Sub-apps build independently:

```bash
cd animal-hospital && npm ci && npm run build       # → animal-hospital/dist
cd hotel && npm install && npm run build            # → hotel/dist
cd movie-theater && npm install && npm run build    # → movie-theater/dist
```

## Deployment

- Pushing to `main` triggers `.github/workflows/deploy.yml`, which:
  1. Builds the main app into `dist/`
  2. Builds `animal-hospital/` → copied into `dist/hospital/`
  3. Builds `hotel/` → copied into `dist/hotel/`
  4. Builds `movie-theater/` → copied into `dist/movie-theater/`
  5. Deploys the merged `dist/` to GitHub Pages
- Live: https://omygodwin.github.io/seasons-study-app/
- Vite `base` is `/seasons-study-app/` — do not change without updating links

## Routing (non-obvious)

Hash-based in `src/App.jsx`:

- `#` (root) → study guides (default)
- `#tournament` → basketball tournament hub (`TournamentApp`)

Study topics are nav-state, not hash-based. The "Rose" dropdown groups
per-child items; add new ones by appending to `ROSE_TOPICS` in App.jsx.

## Structure

```
src/
  App.jsx                   # Hash route + study nav + Rose dropdown
  SeasonsStudyApp.jsx       # Earth science topic
  EgyptStudyApp.jsx         # Ancient Egypt topic
  RocksStudyApp.jsx         # Rocks & Minerals topic
  VocabStudyApp.jsx         # Rose's vocab flashcards + quiz
  tournament/               # Basketball tournament hub
    TournamentApp.jsx, BracketsView.jsx, ScheduleView.jsx, ...
  data/                     # Tournament data
animal-hospital/            # Independent Vite app → dist/hospital/
hotel/                      # Independent Vite app → dist/hotel/
movie-theater/              # Independent Vite app → dist/movie-theater/
```

### movie-theater specifics

- Data model differs from hotel/animal-hospital: each `schedule/{showId}` has its
  own `seats: {1A, 1B, 2A, 2B, 2C}` map (no global `seats` collection). Today's
  schedule auto-seeds from `FEATURED_DEFAULTS` if empty; manager can regenerate
  via Schedule tab → "Generate Default" (atomically clears old shows first).
- 5 seat statuses: `available | sold | seated | dirty | broken`. `seated` is the
  30-min-early arrival flow — customers tap "🪑 Take Seat" on their own ticket
  card in the Now Playing view to flip `sold` → `seated`.
- Public is the default landing (no login wall). Small `🔑 Staff` button switches
  `mode` to 'staff' for the PIN login flow.
- Login screen falls back to hardcoded `DEFAULT_STAFF` and `DEFAULT_GROUPS`
  constants when Firebase reads return empty — so login stays usable even if
  rules temporarily block writes.

## Study-app conventions

- Each `*StudyApp.jsx` follows the same pattern: tab state, flashcard state
  (Known/Review sets), randomized 10-question quiz from a larger pool.
- When adding a new Rose study topic:
  1. Create `FooStudyApp.jsx` mirroring `VocabStudyApp.jsx`
  2. Add `{ id, label, emoji }` to `ROSE_TOPICS` in `App.jsx`
  3. Add `{studyTopic === 'foo' && <FooStudyApp />}` render branch
- Touch-first nav: nav buttons use `min-h-[44px]`; the Rose dropdown closes on
  outside tap or Escape. Preserve these when editing nav.
- No test framework — verify with `npm run build` and manual browser check.

## Gotchas

- The `animal-hospital` build uses `npm ci` (lockfile-strict); `hotel/` and
  `movie-theater/` use `npm install` because their lockfiles are intentionally
  absent (see commit 83762e3 for hotel; same pattern for movie-theater).
- PWA manifest lives at `public/manifest.json`.
- Shared Firebase Realtime DB project `roseruthclinic` is used by `hotel/`,
  `animal-hospital/`, and `movie-theater/` (under namespace `movieTheater`),
  not by the main study app.
- Firebase RTDB rules (Console → roseruthclinic → Realtime Database → Rules) must
  stay `{".read": true, ".write": true}`. Default test-mode rules use a 30-day
  timestamp expiry that silently breaks ALL writes — watch for `permission_denied`
  warnings in the browser console when seeds/writes don't take effect.
- Worktree gotcha: `main` is owned by the primary worktree at
  `C:/Users/bgodwin/Documents/GitHub/seasons-study-app`. From feature worktrees,
  branch via `git checkout -b foo origin/main` — `git checkout main` errors
  with "already used by worktree".
- Port 5173 is also reserved for other local projects per the workspace
  CLAUDE.md — pin a different port in `vite.config.js` if that conflicts.
- `movie-theater/` dev server pinned to port 5188 in `.claude/launch.json`.

---
> Source: [omygodwin/seasons-study-app](https://github.com/omygodwin/seasons-study-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
