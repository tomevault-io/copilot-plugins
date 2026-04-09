## hexalife

> A wealth-based Game of Life simulation on a hexagonal grid, built with React + TypeScript.

# HexaLife

A wealth-based Game of Life simulation on a hexagonal grid, built with React + TypeScript.

## Project Overview

HexaLife is a cellular automaton inspired by Conway's Game of Life, but with two key differences:
1. **Hexagonal grid** - Each cell has 6 neighbors instead of 8
2. **Wealth-based rules** - Instead of simple alive/dead states with probability-based transitions, cells have a wealth value that determines their survival

## Game Rules

### Wealth System

Each alive cell has two properties:
- **Age**: Number of generations the cell has been alive (starts at 1)
- **Wealth**: Numeric value representing the cell's resources (starts at `DEFAULT_STARTING_WEALTH = 10`)

### Wealth Rules

The game uses a `WealthRules` array of 7 numbers (indices 0-6, representing neighbor counts). Each value represents the profit or cost for having that many neighbors:
- **Positive value** = Profit (cell gains wealth)
- **Negative value** = Cost (cell loses wealth)

Default rules: `[-1, -3, -1, 1, -1, -2, -3]`
- 0 neighbors: -1 (isolated cells slowly lose wealth)
- 1 neighbor: -3 (worst case)
- 2 neighbors: -1 (still losing)
- 3 neighbors: +1 (only profitable configuration)
- 4+ neighbors: negative (overcrowding)

### Cell Lifecycle

**Living cells:**
- Each generation: `newWealth = currentWealth + wealthRules[neighborCount]`
- If `newWealth > 0`: Cell survives with `age + 1`
- If `newWealth <= 0`: Cell dies

**Dead cells:**
- A dead cell becomes alive if `wealthRules[neighborCount] > 0` (profitable position)
- New cells start with `DEFAULT_STARTING_WEALTH` (10)

### Simulation Behavior
- Simulation automatically stops when all cells die
- Generation counter tracks the current iteration
- State is persisted to localStorage

## Project Structure

```
src/
├── App.tsx                 # Main app component, orchestrates everything
├── index.css               # Global styles
├── main.tsx                # Entry point
├── types.ts                # TypeScript type definitions
├── gameLogic.ts            # Core game logic (pure functions)
├── gameLogic.test.ts       # Game logic tests
├── components/
│   ├── HexGrid.tsx         # Canvas-based hex grid renderer
│   ├── HexGrid.test.tsx
│   ├── Controls.tsx        # Simulation controls UI
│   ├── Controls.test.tsx
│   ├── Metrics.tsx         # Stats display (generation, cells, wealth)
│   ├── Metrics.test.tsx
│   ├── WealthRulesPopup.tsx    # Modal for editing wealth rules
│   └── WealthRulesPopup.test.tsx
└── hooks/
    ├── useGameOfLife.ts    # Main game state hook
    └── useGameOfLife.test.ts
```

## Key Types

```typescript
// Cell data structure
interface CellData {
  age: number;    // 0 = dead, 1+ = generations alive
  wealth: number; // current wealth (only meaningful when alive)
}

// Wealth rules array (index = neighbor count, value = profit/cost)
type WealthRules = number[];

// Grid state
interface GridState {
  cells: CellData[][];
  width: number;
  height: number;
}

// Coloring modes for visualization
type ColoringMode = 'age' | 'neighbors' | 'wealth';

// Simulation states
type SimulationState = 'stopped' | 'paused' | 'playing';
```

## Coloring Modes

All coloring modes use **relative scaling** based on the current grid state:

1. **Age** - Green (youngest cell in grid) to Red (oldest cell in grid)
2. **Neighbors** - Blue (0 neighbors) to Yellow (6 neighbors) - fixed scale
3. **Wealth** - Red (poorest cell in grid) to Cyan (richest cell in grid)

The relative scaling means colors automatically adjust as the simulation runs, always showing the full color range for the current population.

## Key Files

### `gameLogic.ts`
Pure functions for game mechanics:
- `createEmptyGrid()` / `createEmptyCell()` / `createAliveCell()`
- `computeNextGeneration()` - Main simulation step
- `getNeighborCount()` - Counts alive neighbors (hex grid)
- `toggleCell()` / `setCell()` - Cell manipulation
- `resizeGrid()` / `randomizeGrid()` - Grid operations

### `useGameOfLife.ts`
React hook managing:
- Grid state and simulation state
- Speed and wealth rules settings
- Generation counter
- localStorage persistence
- Simulation loop (play/pause/step)
- Auto-stop when all cells die

### `HexGrid.tsx`
Canvas-based renderer:
- Draws hexagonal cells with flat-top orientation
- Handles mouse interactions (click to toggle, drag to draw)
- Shows tooltip with age/wealth on hover
- Supports three coloring modes

## Development Commands

```bash
# Install dependencies
npm install

# Run development server
npm run dev

# Run tests
npm test

# Run tests once (no watch)
npm test -- --run

# Build for production
npm run build
```

## localStorage

State is automatically saved to `localStorage` under key `hexalife-state`:
```json
{
  "gridState": { "cells": [...], "width": 60, "height": 60 },
  "wealthRules": [-3, -2, 2, 3, -1, -2, -3],
  "speed": 200
}
```

## Presets

WealthRulesPopup offers preset configurations:
- **Classic**: `[-5, -3, 3, 5, -2, -4, -5]` - Survives best with 2-3 neighbors
- **Stable**: `[-2, 0, 2, 2, 0, -1, -2]` - Slow, stable growth
- **Volatile**: `[-10, -5, 5, 10, -3, -6, -10]` - High risk, high reward
- **Social**: `[-5, -2, 1, 2, 3, 1, -1]` - Benefits from more neighbors

## Hex Grid Geometry

- Flat-top hexagons
- Odd rows are offset by half a hex width
- Each cell has up to 6 neighbors (accounting for edge cases)
- Neighbor positions depend on whether the row is odd or even

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sztanko)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sztanko)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
