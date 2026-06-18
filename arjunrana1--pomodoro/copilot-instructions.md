## pomodoro

> Orientation for Claude. The goal: jump directly to the right file without re-reading the codebase. If a section below covers your task, trust it and edit; only read other files when explicitly told "in doubt, verify."

# CLAUDE.md — Pomodoro Focus

Orientation for Claude. The goal: jump directly to the right file without re-reading the codebase. If a section below covers your task, trust it and edit; only read other files when explicitly told "in doubt, verify."

---

## Repo layout

```
/                          ← repo root (docs, PRD, CHANGELOG live here)
  PRD.md                   ← canonical product spec — source of truth for behavior + copy
  DESIGN.md                ← color palette, typography tokens, glassy aesthetic
  design_tokens.md         ← extended design system reference
  CHANGELOG.md             ← released-version log
  test-report.txt          ← manual QA evidence (53 test cases for v2)
  reference_designs/       ← PNG mockups, including v2/
  design_assets/           ← raw design source files
  test-evidence/           ← screenshot outputs from manual QA runs (gitignored beyond reports)
  app/                     ← the actual Vite + React app
```

All code lives under `app/`. Outside of `app/`, files are docs/assets only.

```
app/
  package.json             ← npm scripts (dev / build / lint / preview)
  vite.config.ts           ← Vite + Tailwind v4 plugin config
  index.html               ← SPA entry; mounts main.tsx
  src/
    main.tsx               ← React root, mounts <App />
    App.tsx                ← Top-level router by state.status (idle/running/paused/complete)
    App.css                ← Global one-off styles (glass utilities, animated bg)
    index.css              ← Tailwind directives + base resets
    store.ts               ← THE single source of truth for app state (useAppState hook)
    types.ts               ← All TypeScript types (AppState, PlanItem, NoteItem, DayRecord, FocusHistory)
    utils.ts               ← Pure helpers (formatTime, formatHours, date keys, history math)
    audio.ts               ← WebAudio oscillator-based sound effects
    assets/                ← Static image assets imported by components
    components/            ← All UI components (see map below)
```

---

## Component map — UI editing index

Each row tells you exactly which file owns what visual surface. **Edit only the file listed.** Files are short (most under 250 lines) — when you open one, you have the whole feature in context.

| Surface / Feature | File | Notes |
|---|---|---|
| Home screen layout, brand mark, preset pills, custom-duration pill, daily stats, marketing footer | `components/HomeScreen.tsx` | Footer copy is at the bottom (sections: What is, Pomodoro Technique, How to Use, Features). The marketing footer copy is canonical-spec'd in `PRD.md §7.7`. |
| Active session orb, timer display, Pause/Resume/Stop controls, locked left rail, plan-task checklist during session | `components/ActiveSession.tsx` | The orb's shimmer pauses via `animationPlayState`. Task circles are 20 px with 2.5 px primary border. |
| Flow Complete modal (post-session summary), session focus time, tasks accomplished, session notes, New Session CTA | `components/FlowComplete.tsx` | Shows `lastSessionElapsedSeconds` (stopped) or `initialSeconds` (natural). |
| Session Plan drawer (left side) — task title input, minutes input, Add Task button, drag-reorder list, Start Focused Session CTA | `components/SessionPlanSidebar.tsx` | Terminology must be "Task" / "Add Task" — never "Sub-task" (PRD §12). |
| Notes drawer (right side) — note input, edit/delete list, click-outside-to-close | `components/NotesDrawer.tsx` | Click-outside handler at lines ~53–66. |
| Focus History dashboard — 7-day bar chart, 2×2 stats grid, hourly heatmap, empty state | `components/FocusHistoryDashboard.tsx` | Reads `focusHistory.days`. Bar chart uses `formatHoursDecimal`. Heatmap is 24 hourly columns, 5 intensity tiers in `intensityClass()`. |
| App brand mark (logo SVG) | `components/BrandMark.tsx` | 35×35 SVG. Used in HomeScreen + ActiveSession headers. |

The 7-component list is complete — there is nothing else under `components/`.

---

## State, persistence, audio — non-UI but UI-adjacent

| What | File | Key exports / shapes |
|---|---|---|
| App state hook + all mutations | `store.ts` | `useAppState()` returns `{ state, selectPreset, commitCustomMinutes, startSession, pauseSession, stopSession, addNote, ... }`. Timer is wall-clock-driven (`computeRemainingFromWallClock`). |
| All TypeScript types | `types.ts` | `AppState`, `PlanItem`, `NoteItem`, `DayRecord`, `FocusHistory`, `SessionMode`, `AppStatus`. Add fields here first, then thread through `store.ts`. |
| Pure helpers | `utils.ts` | `formatTime`, `formatHours`, `formatHoursDecimal`, `getTodayKey`, `lastNDates`, `dayInitial`, `migrateDayRecord`, `recordFocusedSeconds`, `emptyDayRecord`. Hour-bucket math lives here. |
| Sound effects | `audio.ts` | `playClickSound`, `playStartSound`, `playPauseSound`, `playResumeSound`, `playStopSound`, `playCompletionSound`. All gated on `state.soundEnabled` in `store.ts`. |

### localStorage keys (current)

- `pomodoro-focus-state` — full app state minus history (set by `persistState`)
- `pomodoro-focus-stats` — `{ dateKey, focusSeconds, sessionsCount }` for today
- `pomodoro-focus-history` — `FocusHistory.days` keyed by `YYYY-MM-DD`

Legacy keys (`deep-focus-state`, `deep-focus-stats`) are migrated on first load via `migrateLegacyKeys()` in `store.ts`. Don't reintroduce the old names.

---

## Common UI change recipes

These cover ~90% of likely follow-up edits. Follow the recipe, don't re-explore.

### "Change a label / piece of copy"
1. Find the surface in the component map above.
2. Edit the JSX text directly.
3. If the copy is product-spec'd (footer marketing sections, brand name, rail labels, "Flow Complete"), also update `PRD.md` so spec ↔ implementation stay aligned.

### "Change how a duration is displayed"
- `utils.ts` owns all `format*` helpers. Add a new helper here rather than inlining formatting in components.
- Then swap the import in the relevant component (usually `FocusHistoryDashboard.tsx`, `HomeScreen.tsx`, `ActiveSession.tsx`, or `FlowComplete.tsx`).

### "Change the dashboard look (bar chart / stats grid / heatmap)"
- Single file: `components/FocusHistoryDashboard.tsx`.
- Stats tile labels live in the JSX directly (e.g. `<StatTile label="Daily Avg" ...>`).
- Bar/heatmap colors come from inline `style={{ background: ... }}` blocks or the `intensityClass()` function near the top — they're not in CSS files.

### "Tweak the active-session orb (controls, task list, timer text)"
- Single file: `components/ActiveSession.tsx`.
- Orb shimmer + paused state at line ~98–104.

### "Change Home screen layout / first-fold elements"
- `components/HomeScreen.tsx`. The file is two halves:
  - Lines ~74–240: the full-viewport "first fold" with timer orb.
  - Lines ~240+: below-fold sections — `<FocusHistoryDashboard />` then `<footer>` marketing.

### "Update marketing footer copy or FAQ-style content"
- `components/HomeScreen.tsx`, the `<footer>` block at the bottom.
- The canonical text is in `PRD.md §7.7`. Update both, or update PRD first and copy.

### "Add a new global state field"
1. Add the field to `AppState` in `types.ts`.
2. Add a default to `getDefaultState()` in `store.ts`.
3. Add it to `toSave` in `persistState()` if it should survive a refresh.
4. Add a mutator (e.g. `setX`) in `useAppState`, return it from the hook.
5. Thread it down through `App.tsx` props.

### "Add a new sound"
1. Add a `play*Sound` export in `audio.ts` (use `playTone(freq, duration, type, volume)`).
2. Gate the call on `prev.soundEnabled` in `store.ts` (search for `playPauseSound` for the pattern).

### "Change the timer logic"
- `store.ts` — sole owner. Look for `computeRemainingFromWallClock` (line ~110) and the `useEffect` at line ~263 (the tick loop).
- Don't add a setInterval in a component. The store is the only place that holds timer state.

### "Change a color / glass effect / animation"
- App-level CSS: `src/App.css` (glass utilities, animated backgrounds, shimmer).
- Tailwind tokens: edit class names inline. Tailwind v4 is configured via `vite.config.ts` (no `tailwind.config.js`).
- Brand palette: documented in `DESIGN.md`. Primary purple is `#6A5AE7` (hex) / `bg-primary` (Tailwind alias).

---

## Build / dev / lint commands

Run from the repo root or `app/` — both work because npm scripts are defined in `app/package.json`.

```
cd app
npm run dev      # local dev server (Vite)
npm run build    # production build (tsc -b && vite build)
npm run lint     # eslint
npm run preview  # serve the production build locally
```

A successful production build produces `app/dist/`. Vercel deploys from this — Vercel project root must be set to `app/`.

---

## Workflow conventions

- **Branch off `main`** for every change: `git checkout -b feature/<thing>` or `chore/<thing>` or `fix/<thing>`.
- **Open a PR via `gh pr create`** — full path is `~/.local/bin/gh` unless PATH is updated. The `gh` token in macOS Keychain is already auth'd as `arjunrana1`.
- **GitHub auto-deletes merged head branches** (verify the setting is on, otherwise: `git push origin --delete <branch>` after merge).
- **Don't commit `.DS_Store`** — `.gitignore` handles macOS noise. If a stray one is staged, unstage it.
- **PRD.md is the spec** — when behavior or copy changes, update PRD in the same PR. Reviewers (including future Claude) read it first.
- **Don't add documentation files** (`*.md`, READMEs) unless explicitly asked. CLAUDE.md, CHANGELOG.md, PRD.md, DESIGN.md, and `app/README.md` are the only sanctioned ones.

---

## What NOT to read by default

To stay token-efficient, skip these unless your task explicitly requires them:

- `test-report.txt` (long, historical QA log — only read if debugging a regression)
- `test-evidence/` (binary PNGs, useless to LLMs)
- `reference_designs/`, `design_assets/` (binary)
- `node_modules/`, `dist/` (always skip)
- `package-lock.json` (only if dependency conflict)
- Old PR/branch state — git history rarely needs reading for UI changes

If a single component file plus its imports gives you the answer, stop there.

---

## When in doubt

1. **Behavior question?** → `PRD.md` is the source of truth.
2. **Visual / color / spacing question?** → `DESIGN.md` or `design_tokens.md`.
3. **"Where is X rendered?"** → component map above; only grep if it's truly missing.
4. **"Why does the timer do Y?"** → `store.ts` is the only place with timer logic.
5. **Type / shape question?** → `types.ts`.

If two files seem to compete for ownership of a feature, the component map wins — that's the canonical assignment. Flag any drift back to me in the PR description.

---
> Source: [arjunrana1/pomodoro](https://github.com/arjunrana1/pomodoro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
