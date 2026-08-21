## react-gameroom

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- **Build:** `npm run build` (uses tsup, outputs CJS/ESM/types to `dist/`)
- **Type check:** `npm run typecheck` (runs `tsc --noEmit`)
- No test runner or linter is configured.

## Architecture

This is a **published React component library** (not an app) for multiplayer game lobby management. It exports components, hooks, helpers, and types via barrel exports from `src/index.ts`. Consumers manage state externally — the library is stateless by design.

### Core domain model (`src/types/room.ts`)

- **RoomState**: contains `roomId`, `status` (lobby/started), `players` (fixed-length array of `PlayerSlot`), and `config` (min/max players, requireFull flag)
- **PlayerSlot**: has `id` (1-based) and `status` (empty → joining → ready)

### Layer separation

- **`types/`** — domain types only
- **`helpers/roomHelpers.ts`** — pure state transition functions (`createInitialRoom`, `setPlayerJoining`, `setPlayerReady`, `resetPlayer`, `startGame`). All return new objects (immutable). Helpers validate preconditions via guard clauses.
- **`utils/roomUtils.ts`** — URL generation/parsing and room ID generation. URL pattern: `/room/{id}` and `/room/{id}/player/{n}`
- **`hooks/useRoomState.ts`** — `useRoomState(state)` returns `RoomDerivedState` with computed fields (`canStart`, `readyCount`, etc.)
- **`components/`** — presentational only, prop-driven with callback-based updates. No internal state management.

### Component hierarchy

`PlayerSlotsGrid` → `PlayerSlotView`. `PlayerScreen` is the mobile/player view with render props (`renderStarted`, `renderEmpty`, `renderReady`). `RoomQRCode` wraps qrcode.react. Consumers compose these primitives into their own layouts.

## Conventions

- TypeScript strict mode, ES2018 target, bundler module resolution
- All component prop interfaces use `*Props` suffix
- Peer dependency on React 17+; library ships CJS + ESM + `.d.ts`

---
> Source: [raphaelaleixo/react-gameroom](https://github.com/raphaelaleixo/react-gameroom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
