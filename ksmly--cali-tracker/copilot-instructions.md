## cali-tracker

> A mobile-first PWA for tracking calisthenics skill progression. Helps freestyle trainers develop consistency and balance naturally — through visibility, not prescription.

# Calisthenics Progress Tracker

A mobile-first PWA for tracking calisthenics skill progression. Helps freestyle trainers develop consistency and balance naturally — through visibility, not prescription.

**Full product spec:** `docs/PRODUCT_SPEC.md`

---

## Stack

- **React + Vite** — UI framework
- **TypeScript** — strict mode enabled
- **Tailwind CSS** — styling (no separate CSS files)
- **Recharts** — charts and data visualization
- **localStorage** — storage v1, accessed only through `src/services/db.ts` (never directly from components)
- **Vite PWA plugin** — service worker, offline support, installability
- **GitHub Pages** — hosting via `gh-pages` branch

---

## Project Structure

```
src/
  components/       # Reusable UI components
  pages/            # Top-level route pages (Dashboard, LogSession, SkillDetail, etc.)
  services/
    db.ts           # ALL storage read/write goes through here — never use localStorage directly
    suggestions.ts  # "What to train today" logic
    streaks.ts      # Streak calculation logic
  data/
    skills.ts       # Built-in skill definitions and progression ladders
    exercises.ts    # Built-in exercise library
  types/            # All TypeScript types and interfaces (source of truth)
    index.ts
  hooks/            # Custom React hooks
  utils/            # Pure helper functions
  theme/            # Theme definitions and CSS variables
docs/
  PRODUCT_SPEC.md   # Full product specification
```

---

## Core Data Types

See `src/types/index.ts` for the full definitions. Key types:

- `Skill` — a movement with a progression ladder
- `SkillStage` — one step on a skill's ladder
- `Exercise` — a drill (reps-based or hold-based)
- `Session` — a logged workout with date, entries, notes
- `ExerciseEntry` — sets × reps or sets × seconds within a session
- `ProgressionEvent` — a skill level-up milestone
- `UserSettings` — theme, streak config, etc.

---

## Key Conventions

- **Never access localStorage directly.** Always use `src/services/db.ts`. This keeps the storage layer swappable.
- **TypeScript strict mode.** No `any`. If you're unsure of a type, define it in `src/types/index.ts`.
- **One component, one job.** Keep components focused. Logic goes in hooks or services, not JSX.
- **Mobile-first.** All layouts start from mobile. Use Tailwind's `md:` prefix for desktop adjustments.
- **Feature branches.** Never commit directly to `main` or `dev`. Branch name format: `feature/<short-description>`.
- **Commits are checkpoints.** Commit before asking Claude to make significant changes. Use present tense: `add workout logger`, not `added workout logger`.

---

## Branch Strategy

```
main        ← production only, deployed to GitHub Pages
dev         ← integration branch, merge features here first
feature/*   ← one branch per feature, short-lived
```

---

## Running Locally

```bash
npm install
npm run dev
```

---

## Deploying

```bash
npm run build
npm run deploy   # deploys dist/ to gh-pages branch
```

---

## Current Phase

> Update this section as the project progresses.

- [x] Scope defined
- [x] Data model finalized (`src/types/index.ts`)
- [x] Project scaffolded (Vite + React + TS + Tailwind)
- [ ] GitHub repo created with branch strategy
- [ ] CI deploy to GitHub Pages working
- [ ] Onboarding flow
- [ ] Dashboard
- [ ] Session logger
- [ ] Skill detail view
- [ ] Charts (Recharts)
- [ ] Streaks + notifications
- [ ] Themes
- [ ] Export / import

---
> Source: [KSMLY/cali-tracker](https://github.com/KSMLY/cali-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
