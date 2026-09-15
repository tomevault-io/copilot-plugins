## matelab

> Read the exact versioned docs at https://docs.expo.dev/versions/v54.0.0/ before writing any code.

# MateLab — Agent Guide

## Expo HAS CHANGED

Read the exact versioned docs at https://docs.expo.dev/versions/v54.0.0/ before writing any code.

## What this is

MateLab is a chess app: play against a bundled Stockfish engine, analyze games, solve puzzles, and track stats. Built on Expo SDK 54 / React Native 0.81 with the New Architecture enabled (`newArchEnabled: true`).

## Commands

```bash
npm start              # expo start (dev server)
npm run ios            # build + run iOS dev client
npm run android        # build + run Android dev client
npx tsc --noEmit       # typecheck (no other lint/test tooling is configured)
```

This app requires a **dev-client build, not Expo Go** — it bundles a native Webview + Stockfish WASM. After changing native deps or `app.json`, run `npx expo prebuild` and rebuild.

## Architecture

**No router.** `App.tsx` owns a `tab` state and manually renders one of five screens (Play, Analysis, Puzzles, History, Stats). There is no react-navigation or expo-router.

**Engine = hidden WebView running Stockfish WASM.** This is the least obvious part of the codebase:

- `assets/engine/` holds `engine.html`, the engine-glue JS (`engine-glue.txt`), and `stockfish-18-lite-single.wasm`. Metro must serve these, which is why `metro.config.js` pushes `html`, `wasm`, `txt` into `assetExts` — don't remove those.
- `App.tsx` renders one hidden `<WebView>` at the root and a single `EngineService` instance shared by Play and Analysis screens.
- `src/engine/EngineService.ts` owns the lifecycle (attach → boot → UCI handshake → `waitReady()`), allows exactly **one in-flight search**, and enforces a movetime watchdog. `src/engine/protocol.ts` is the serialized message contract between RN and the WebView — keep payloads flat strings; if you change one side, change both.

**All persistence goes through `src/storage/db.ts`.** It opens `matelab.db` via `expo-sqlite` (WAL mode) with three tables: `games` (PGN/FEN/result/accuracy per game), `puzzle_progress`, and `stats` (generic key/value). Screens never touch SQLite directly — add query helpers there, following the existing `runAsync`/`getAllAsync` patterns.

**Game rules come from `chess.js`.** Move generation, legality, and PGN/FEN handling are `chess.js`'s job; `src/game/useChessGame.ts` wraps it as the stateful hook used by the play/analysis UIs, and `src/game/clock.ts`, `pgn.ts`, `review.ts` stay rules-adjacent only.

**Stats and coach are pure functions.** `src/stats/compute.ts` and `src/coach/insights.ts` take `StoredGame[]` and return summaries/tips — no I/O inside. Keep them pure so they stay testable and reusable.

## Conventions

- TypeScript everywhere; screens/components are `.tsx`, logic modules are `.ts`. No default-props, no class components.
- Components live in `src/components/`, full-screen views in `src/screens/`. A screen that needs the engine receives the shared `EngineService` as a prop — never construct a second one.
- Data flow for reads is the plain `useEffect` → async query → `setState` pattern (see `StatsScreen`); there is no react-query/zustand/redux.
- **All UI colors come from `src/theme/`.** `useTheme()` returns semantic tokens (`bg`, `card`, `accent`, …) for the active light/dark palette; build per-component styles with `useMemo(() => makeStyles(colors), [colors])`. Never hardcode hex values in components — the exceptions are the fixed game surfaces (board squares/pieces/overlays in `Board.tsx`, eval-bar white/black fills, promotion piece glyphs), which stay constant across themes. Appearance follows the OS (`userInterfaceStyle: automatic`) with a persisted system/light/dark pick stored via `getSetting('theme')`/`setSetting` in `src/storage/db.ts`; the toggle lives in the Play header.
- Product intent and feature ideas live in `docs/ideas/matelab.md` — consult it before redesigning flows.

## Gotchas

- Accuracy fields are nullable (`whiteAccuracy`/`blackAccuracy`) and `playerColor` is null for puzzles — stats code must keep handling those cases.
- `StoredGame.result` uses PGN results: `"1-0" | "0-1" | "1/2-1/2" | "*"` where `"*"` means unfinished.
- `android/` and `ios/` are checked-in prebuild outputs; regenerate with `npx expo prebuild` rather than hand-editing, unless the change is something prebuild would wipe anyway.

---
> Source: [tarunrajput/matelab](https://github.com/tarunrajput/matelab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-15 -->
