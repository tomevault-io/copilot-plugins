## staircasetrial

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

StaircaseTrial (楼道暴露挑战) is a web-based game where the player climbs a staircase completing randomly assigned challenges, manages points (积分) and clothing items, and reaches one of several endings based on final score. The entire UI and game content are in Chinese. It was migrated from a vanilla JS version (kept in `legacy/`) to React + TypeScript in v2.0.

## Commands

```bash
npm install        # install dependencies
npm run dev        # start Vite dev server
npm run build      # typecheck (tsc -b) + production build to dist/
npm run preview    # preview the production build
npm run lint       # eslint . (flat config)
```

There are no tests configured. Deployment is automatic: pushing to `main` triggers `.github/workflows/deploy.yml`, which builds and deploys to GitHub Pages (Vite `base: '/StaircaseTrial/'`).

## Architecture

The key design decision: **all game logic lives in `src/lib/game/` as framework-free TypeScript**, and React components are thin renderers over a Zustand store. UI logic and game rules should be implemented there, not in components.

### `src/lib/game/` — core game logic

- `types.ts` — `GameState`, `Task`, `ClothingItem`, `AppView`, `GameHistoryRecord`, `ShopItem`, `Persona`（`"male" | "female"`，公狗/母狗）。`ClothingItem` is a Chinese-literal union type (`"上衣" | "长裤" | "内裤" | "短袜" | "护膝"`).
- `constants.ts` — all game content and pure functions: male task pools (`tasks.A/B/C/上楼任务/H`), `getTasks(persona)`, `endings`, floor maps, and helpers (`createInitialGameState`, `calculateTaskScore`, `getEnding`, `pickRandomTask`, `getTaskPoolForFloor`, `getDisplayFloor`, `getProgressPercent`).
- `tasksFemale.ts` — 母狗模式任务与结局（按女生改写，含掰穴、勒缝、奶子贴墙等）。
- `storage.ts` — localStorage persistence. `gameState` key holds the current game, `staircaseGameHistory` holds up to 10 history records.
- `store.ts` — the Zustand store `useGameStore`: `state`, `view`, `history`, plus all game actions (`startGame`, `buyItem`, `startAdventure`, `completeTask`, `nextFloor`, `assignClimbingTask`, `confirmClimbing`, `skipTask`, `useRestoreVoucher`, `forfeitGame`, `endGame`, `restartGame`). Actions call `persist()` (save to localStorage) explicitly after every mutation — there is no zustand persistence middleware. Also exports pure UI helpers `getGameControls` and `getActiveTaskDisplay` used by `GameSection`.

### Game flow (the state machine)

`GameState.gamePhase` transitions `initial → shop → adventure → ended`; the store also tracks a separate `view` (`start | rules | shop | game | end | history`) that `App.tsx` renders via section components in `src/components/sections/`.

- **Internal vs display floors**: `currentFloor` is relative to the chosen starting floor (internal range 1–10). `getDisplayFloor(floor, startingFloor)` converts for display.
- **Shop**: player buys items with starting points. `startAdventure` refuses unless `长裤` (pants, the required item) is owned.
- **Task phases**: floors 1–2 draw from pool A, 4–5 from pool B, 7–8 from pool C. After completing the tasks on a phase floor, the player moves to the "climbing decision" floors (2, 5, 8) via `NEXT_FLOOR_MAP`, where `assignClimbingTask` assigns an `上楼任务` with a `targetFloor` from `CLIMBING_TARGET_MAP` (2→4, 5→7, 8→10); `confirmClimbing` advances there. Floor 8 is special: it requires two tasks per visit (`eighthFloorFirstTaskCompleted`).
- **Scoring**: `calculateTaskScore` = task `baseScore` + bonus for each missing key clothing item (fewer clothes worn → more points). Completing a task also has a ~50% random strip event (dice ≤ 3): a random worn **strippable** item is removed unless the player pays 5 points to keep it. 护膝 is never stripped.
- **Progress**: `progressStepsCompleted` (out of `TOTAL_PROGRESS_STEPS = 10`) drives the progress bar; reaching the top floor calls `endGame`, which records a history entry and computes the ending from `getEnding(score)` thresholds (normal 40/30/20/10/1/0, hell 60/45/30/15/0).
- **Confirmations**: async operations that need user input (e.g. paying to keep clothing, skipping, forfeiting) use `requestConfirm(message, options)` — a promise-based bridge to the `ConfirmDialog` component, which resolves via `resolveConfirm`.

### Components

- `src/components/sections/` — one component per view (`StartSection`, `RulesSection`, `ShopSection`, `GameSection`, `EndSection`, `HistorySection`).
- `src/components/AppHeader.tsx` — title-click activates the hidden debug mode (10 clicks; see `src/hooks/useDebugMode.ts`), which monkey-patches `console.*` and shows the `DebugTerminal` with a log buffer.
- `src/components/ui/` — shadcn/ui components (new-york style, zinc base color); add new ones via `npx shadcn@latest add <name>`.
- `@/` imports alias to `src/` (configured in `vite.config.ts`).

## Conventions & notes

- **Content is data, not code**: adding tasks, items, or endings means editing `constants.ts` / `tasksFemale.ts` — `tasks`, `tasksFemale`, `shopItems`, `endings`. Prices/balances are tuned there too (e.g. skip = 6 pts, restore = 8 pts, risk double = 6 pts).
- `rules.md` at the repo root is the game design spec (Chinese). It describes the intended v2 rules, but the code is the source of truth where they differ.
- `legacy/` holds the pre-migration vanilla JS game for reference — not built, do not modify unless porting something from it.
- All user-facing strings are Chinese; keep new UI text and content in Chinese.

---
> Source: [hulifoxi/StaircaseTrial](https://github.com/hulifoxi/StaircaseTrial) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
