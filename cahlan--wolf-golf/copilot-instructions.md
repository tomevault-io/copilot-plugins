## wolf-golf

> Guidance for working in this repository.

# CLAUDE.md

Guidance for working in this repository.

## What this is

**Wolf Tracker** (`wolf-app`) — a mobile-first web app for tracking side-bet golf games
among a foursome. Supports three game types: **Wolf** (classic), **6x6x6**, and **3-2-1**.
State lives in the scorekeeper's `localStorage` and syncs to **Supabase** for multiplayer
join/spectator mode and cross-round weekend leaderboards.

Stack: **Next.js 16** (App Router) · **React 19** · **TypeScript** · **Tailwind v4** ·
**Supabase** (`@supabase/ssr`). Node 22.

## Commands

```bash
npm run dev          # next dev on :3000
npm run build        # next build
npm run lint         # eslint (next core-web-vitals + typescript)
npm run test:unit    # vitest — engine unit tests (src/**/__tests__/**/*.test.ts)
npm test             # playwright e2e (auto-starts next dev; sets LD_LIBRARY_PATH for libasound)
npm run test:ui      # playwright interactive UI mode
```

E2E setup on headless servers may need `bash scripts/install-test-deps.sh` (extracts
libasound to /tmp). See `TESTING.md`. `CI=true` enables retries + no server reuse.

## Architecture

Clean split between a **pure engine** and the **UI / persistence** layers.

```
src/
├── lib/
│   ├── engine/          Pure, framework-free scoring logic — the heart of the app.
│   │                    Public API is re-exported from engine/index.ts; import from there.
│   ├── types/game.ts    All domain types (Game, CompletedHole, Settlement, …).
│   ├── storage/local.ts localStorage persistence (active game, weekend cache, spectator id).
│   └── supabase/        client.ts, courses.ts, weekends.ts — multiplayer + shared weekend.
├── providers/game-provider.tsx   React context: game state, weekend games, and the
│                                  scorekeeper / spectator role split. Auto-saves to
│                                  localStorage + upserts to Supabase (scorekeeper only).
├── app/                 App Router pages: page (home), create, game/[gameId],
│                        game/[gameId]/settlement, courses, weekend/[weekendId], join.
└── components/          create/ wizard steps, game/ hole-input flows + result/standings
                         views, course/ editor, ui/ primitives (button, bottom-sheet, …).
```

### The engine (`src/lib/engine/`)

All per-hole scoring flows through the game-type-aware dispatcher
**`calculateGameHolePoints(game, hole)`** in `scoring-utils.ts`, which delegates by
`game.gameType` (`'wolf'` default | `'six'` | `'three-two-one'`):

- **Wolf** — `wolf.ts` picks the wolf for a hole; `scoring.ts` scores the hole.
  Lone-wolf wins pay **+4 / +3 / +2** (early / late / default declaration —
  see `LONE_WOLF_POINTS` in `constants.ts`). Supports a **wrap-around starting hole**
  (`startingHole`) and a **last-place-wolf** rule (`lastPlaceWolf`, default on from
  `lastPlaceWolfStartHole`=17): the trailing player becomes wolf on the closing holes.
  `wolf.ts` derives round position from the course hole to handle wrap-around.
- **6x6x6** — `six.ts`. Three 6-hole segments, each with a rotating pair; hi/low
  ("low ball" + "high ball") scoring. Teams come from `game.teamSegments`.
- **3-2-1** — `three-two-one.ts`. 3-player mode; rank by net each hole (3/2/1 pts) with
  tie-splitting that always sums to 6 per hole.

Support modules: `handicaps.ts` (stroke allocation by stroke index vs. lowest handicap),
`tee-order.ts` (wolf tees first, others follow in wolfOrder), `standings.ts`, `skins.ts`
(optional carryover), `settlement.ts` (money: 4 payout structures + greedy debt
simplification), `weekend.ts` (multi-round placement points 4/3/2/1),
`matchup.ts` / `*-result` helpers (display strings).

## Conventions

- **Engine is pure.** Keep `src/lib/engine/` free of React, DOM, storage, and Supabase.
  It takes plain `Game`/`CompletedHole` data and returns plain results. Put new scoring
  rules here with a unit test in `engine/__tests__/`.
- **New game type?** Add it to `GameType`, then extend the dispatcher in `scoring-utils.ts`
  (and `settlement.ts`, standings/display as needed) rather than branching in the UI.
- **Advanced-config fields on `Game` are optional with defaults** applied at read time
  (e.g. `game.lastPlaceWolf ?? true`, `startingHole ?? 1`). Preserve that pattern so old
  saved games in localStorage keep working — don't assume a field is present.
- **Persistence is scorekeeper-only.** Spectators receive state via Supabase realtime and
  must never write. Role lives in `game-provider.tsx` (`isScorekeeper` / `isSpectator`).
- **Tests use user-visible selectors** (`getByRole`/`getByText`), not `data-testid`.
  The `?test=1` home-page shortcut loads a pre-built RCC game for e2e (see `TESTING.md`).

## Watch out for

- **`src/app/game/[gameId]/page.tsx` is ~945 lines** — the main orchestration hub and by
  far the largest file. Tread carefully; consider extraction if you add feature there.
- **E2E is marked "non-blocking" in CI** and has failed on recent runs. Don't assume a
  green pipeline means e2e passed — check it explicitly.
- Supabase reads/writes **fail gracefully** by design (dummy creds in tests); code should
  not hard-depend on a live connection for core single-device play.
</content>
</invoke>

---
> Source: [cahlan/wolf.golf](https://github.com/cahlan/wolf.golf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
