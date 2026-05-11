## budget-lines

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SumTrails is a React Native puzzle game where players draw paths through numbered cells to reach a target sum with a minimum line length. Built with Expo SDK 54 and React Native 0.81.

## Commands

```bash
# Development
npm start              # Start Expo dev server
npm run ios            # Run on iOS simulator
npm run android        # Run on Android emulator
npm run web            # Run in web browser

# Testing
npm test               # Run all tests
npx jest --testPathPattern=grid  # Run specific test file

# Deploy
npm run deploy-iphone  # Build and deploy to physical iPhone
```

## Runtime flags (env)

- **Unlimited hints**: set `EXPO_PUBLIC_UNLIMITED_HINTS=true` to allow requesting hints repeatedly (the hint button won’t disable after the first use).

## Architecture

### Navigation (Expo Router)
- `app/_layout.tsx` - Root layout with ThemeProvider, IAP init, notifications
- `app/(tabs)/` - Tab navigation: index (daily), practice, stats, settings
- `app/game/` - Full-screen game screens (daily.tsx, practice.tsx)

### Core Game Logic (`src/core/`)
Pure TypeScript functions with no React dependencies. All puzzle logic is tested.

- `types.ts` - Core types: Cell, GameState, Position, CellState
- `puzzleGenerator.ts` - Generates daily (seeded) and practice (random) puzzles
- `constructiveGenerator.ts` - Builds puzzles from valid solution paths (guarantees solvability)
- `pathValidator.ts` - Validates path adjacency and backtracking
- `sumCalculator.ts` - Path sum calculations and validation
- `stuckDetector.ts` - Detects when no valid moves remain
- `hintGenerator.ts` - Generates hints from solution paths

### State Management (`src/stores/`)
Zustand stores for global state:

- `gameStore.ts` - Game state, path drawing, line commitment, hints
- `userStore.ts` - User stats, streaks, premium status

### Components (`src/components/`)
- `GameBoard.tsx` - Main game board with gesture handling (pan gestures via react-native-gesture-handler)
- `GameHUD.tsx` - Target sum display, progress, action buttons
- `Grid.tsx` / `Cell.tsx` - Grid rendering
- `PathOverlay.tsx` - SVG path visualization

### Services (`src/services/`)
Platform-specific implementations use `.native.ts` and `.web.ts` suffixes:
- `iap` - In-app purchases (react-native-iap)
- `notifications` - Push notifications (expo-notifications)
- `analytics` - Event tracking

### Key Patterns

**Daily Puzzle Reset**: Puzzles reset at 8am Eastern Time daily. The `getDailyPuzzleId()` function handles timezone-aware date calculation with DST support.

**Seeded Generation**: Daily puzzles use seedrandom with puzzle ID as seed, ensuring all users get the same puzzle.

**Cell IDs**: Cells are identified by `"row-col"` string format (e.g., `"2-3"`). Use `cellIdFromPosition()` and `positionFromCellId()` for conversion.

**Cell States**: `'available'` (can use), `'spent'` (used in completed line), `'in-path'` (currently being drawn)

**Gesture Handling**: GameBoard uses worklets for smooth gesture tracking. Touch position → cell ID conversion happens in worklet thread via `getCellIdFromPosition()`.

---
> Source: [caden311/budget-lines](https://github.com/caden311/budget-lines) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
