## tre-giorni

> A 3-day Italy trip planner. Next.js (App Router, TypeScript) + Tailwind CSS. See

# Tre Giorni

A 3-day Italy trip planner. Next.js (App Router, TypeScript) + Tailwind CSS. See
`README.md` for what it does and how to run it — this file is conventions for working
in the codebase.

## Architecture

- **Pure logic lives in `lib/*.ts`; React components are thin wrappers around it.**
  Scoring/scheduling (`lib/scoring.ts`, `lib/scheduler.ts`), drag-board state transitions
  (`lib/board.ts`), hours/seasonality parsing (`lib/hours.ts`, `lib/seasonality.ts`), and
  the AI nudge guardrails (`lib/nudge.ts`) are all functions with no React or DOM
  dependency, unit-tested in `lib/__tests__/`. When adding logic, default to putting it in
  `lib/` and testing it there rather than inline in a component.
- **`data/italy.json` is normalized at runtime, not from a static snapshot.** `lib/data.ts`
  calls `normalizeDataset(italyRaw, new Date())` on first access and caches the result, so
  time-sensitive logic (seasonality, day-of-week hours) is always correct for whenever the
  app is actually visited. `data/italy_normalized.json` is a separate, committed snapshot
  for reviewers (`npm run normalize-data` regenerates it) — it is not imported by the app.
- **The AI nudge feature (`app/api/nudge/route.ts`) is a single forced-tool-use call**,
  not a chat loop. It's constrained to the day's current stops + pool candidates only, and
  every response is structurally validated against that candidate set (`lib/nudge.ts`)
  before being applied to the board — a hallucinated or out-of-scope place ID can't reach
  the UI. Rate-limited per IP (`lib/rateLimit.ts`); the limiter is in-memory and per
  serverless instance, a soft deterrent on Vercel, not a hard cap — the real cost backstop
  is a spend limit on the Anthropic account itself, set outside this repo.

## Testing and verification

- `npm test` (Vitest) covers everything in `lib/`. Any new pure logic should get tests
  there before wiring it into a component.
- Drag-and-drop and other real browser interactions are not covered by automated tests in
  this repo — simulated pointer/keyboard drags proved unreliable to script reliably.
  Verify UI changes by running the dev server and checking manually.
- Run `npx tsc --noEmit`, `npm run lint`, and `npm test` after any change before
  considering it done.

## Known gotchas (already solved, don't reintroduce)

- **Drag-and-drop state mutation must happen only in `onDragEnd`, never `onDragOver`.**
  Mutating state on every hover event previously caused a real "Maximum update depth
  exceeded" crash (dnd-kit's continuous rect re-measurement during a drag feeds back into
  React state updates). `DragOverlay` provides drag feedback without needing
  `onDragOver` to touch state.
- **`collisionDetection` on the board's `DndContext` cannot be plain `closestCenter`.**
  Once a day column is emptied, its only droppable is the container div itself, and
  `closestCenter`'s rect-center-distance comparison often loses to individual card rects
  in neighboring columns even when the cursor is visually over the empty column, silently
  breaking drops into it. `Board.tsx` uses a `pointerWithin` → `rectIntersection` →
  `closestCenter` fallback chain instead (the same pattern dnd-kit's own official
  multi-container example uses).
- **Board day schedules only recompute via `recomputeDaySchedule` once a day is marked
  dirty** (actually touched by a drag or AI nudge). Recomputing on every render discards
  `buildItinerary`'s original hours-aware time slotting in favor of a naive back-to-back
  layout, producing false "outside hours" warnings before the user has done anything.

## Deploying

Vercel, zero special config needed (`next.config.ts` is default). Set
`ANTHROPIC_API_KEY` in the Vercel project's environment variables to enable the AI
feature in production, but the app works fully without it.

---
> Source: [tinabanthia/tre-giorni](https://github.com/tinabanthia/tre-giorni) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
