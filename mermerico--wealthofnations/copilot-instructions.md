## wealthofnations

> **CRITICAL**: Always follow this workflow:

# Wealth of Nations - Project Structure Guide

## Development Workflow

**CRITICAL**: Always follow this workflow:

1. **Run Tests First**: Execute `npm run test` to verify all tests pass
2. **Build the Application**: Run `npm run build` to check for TypeScript/build errors
3. **Avoid Running Dev Server**: Do NOT run `npm run dev` - the user will test functionality. Focus on ensuring tests pass and build succeeds.
4. **Playwright Testing**: When running Playwright tests, ensure the HTML report is disabled (e.g., use `--reporter=line`) so that the command exits immediately upon completion instead of hanging to serve the report.

This ensures you catch errors early and avoid debugging blank pages or runtime issues that could have been caught at build time. Let the user handle running the dev server and testing the UI.

## Overview

This project is a web-based implementation of the board game **Wealth of Nations**, a game of industrial production and international trade. The current implementation supports local hotseat multiplayer (players take turns on the same device). The game is built using React, TypeScript, and Vite.

## Technology Stack

- **Frontend Framework**: React 19 with TypeScript
- **Build Tool**: Vite
- **Testing**: Vitest with React Testing Library
- **Styling**: CSS (component-level)
- **Development**: ESLint for code quality

## Project Architecture

### Core Structure

```
### Core Structure

```
.
├── src/
│   ├── types/          # TypeScript type definitions
│   ├── utils/          # Game logic utilities
│   ├── hooks/          # React hooks (game state management)
│   ├── components/     # React components (UI and game elements)
│   ├── pages/          # Page-level components
│   └── tests/          # Integration and unit tests
│       └── integration/ # Complex interaction tests
├── e2e/                # Playwright End-to-End tests
├── public/             # Static assets (flag SVGs)
├── resources/          # Game rules and reference materials
└── config files        # TypeScript, Vite, ESLint configurations
```

## Core Game Concepts

### Game Mechanics (from resources/rules.md)


The game revolves around three main phases that repeat each round:

1. **Trade Phase**: Players buy and sell commodities on markets
2. **Develop Phase**: Players place flags and build industry tiles on the hexagonal board
3. **Produce Phase**: Industries produce commodities based on their configuration

**Key Resources**:
- **Commodities**: Food (yellow), Energy (blue), Labor (red), Ore (gray), Capital (black)
- **Money**: Purple resource used for trading and scoring
- **Victory Points**: Final scoring mechanism

**Key Mechanics**:
- Hexagonal grid-based board
- Industry tiles with dots (production units) and partial dots that combine with adjacent tiles
- Market system with dynamic pricing based on supply/demand
- Bloc system: connected tiles of the same type produce together
- Automation tokens for enhanced production

## Detailed File Structure

### `/src/types/gameState.ts`

**Purpose**: Central type definitions for the entire game state.

**Key Types**:
- `CommodityType`: Union type for the 5 commodities
- `IndustryType`: Union type for the 6 industry types (Farm, Generator, Academy, Mine, Factory, Bank)
- `Player`: Player data including resources, money, loans, flags
- `HexCell`: Individual hex on the board with coordinate and occupant
- `IndustryTile`: Tile instance with type, owner, orientation, active state, automation
- `TileFeature`: Visual/functional features on tile edges/corners (half-dots, third-dots)
- `MarketState`: Stock level and price index for each commodity market
- `GameState`: Complete game state including players, board, markets, phase, turn order

**Design Pattern**: Immutable state structure designed for React useState/useReducer patterns.

### `/src/hooks/useGameEngine.ts`

**Purpose**: Main game state management hook. This is the "engine" that drives the game.

**Key Responsibilities**:
- Maintains the complete `GameState` using React `useState`
- Provides `handleAction()` function for all game actions
- Manages phase transitions (Trade → Develop → Produce)
- Manages turn order and player switching
- Implements game actions:
  - `pass`: Skip turn, advance to next player/phase
  - `placeFlag`: Place a player flag on an empty hex
  - `placeTile`: Build an industry tile on a flagged hex
  - `buy`/`sell`: Market transactions
  - `rotate`/`move`: Manipulate tiles during Develop phase
  - `automate`: Add automation token to a tile
  - `produce`: Execute production for a bloc
  - `delete`: Remove tiles/flags

**State Management Pattern**: 
- Uses functional setState updates for immutability
- Validates actions before applying state changes
- Returns success/failure results for UI feedback

**Key Functions**:
- `nextTurn()`: Cycles to next player
- `nextPhase()`: Advances game phase
- `handleAction()`: Central dispatcher for all game actions

### `/src/utils/` Directory

#### `hexUtils.ts`
**Purpose**: Hexagonal grid mathematics and coordinate system.

**Key Functions**:
- `generateGrid(radius)`: Creates the hexagonal board
- `coordsToString(q, r)`: Converts coordinates to string keys
- `stringToCoords(key)`: Parses coordinate strings
- `getNeighbor(coord, direction)`: Gets adjacent hex in a direction
- `getNeighbors(cell, board)`: Gets all 6 neighbors

**Coordinate System**: Uses cube/axial coordinates (q, r) for hexagonal grid math.

#### `tileDefinitions.ts`
**Purpose**: Static definitions for all 6 industry tile types.

**Key Exports**:
- `TILE_DEFINITIONS`: Object mapping IndustryType to TileDefinition
- Each definition includes:
  - `features`: Array of TileFeature (dots on edges/corners)
  - `hasCenterDot`: Whether tile has a center production dot
  - `cost`: Text description of building cost
  - `costStruct`: Structured commodity costs
- `COMMODITY_COLORS`: Color mappings for visual rendering

**Tile Feature System**: 
- Features are positioned 0-5 (N, NE, SE, S, SW, NW)
- Half-dots on edges combine with adjacent tiles
- Third-dots on corners combine when 3 tiles meet

#### `production.ts`
**Purpose**: Production calculation logic for the Produce phase.

**Key Functions**:
- `identifyBloc(board, startCell)`: Finds all connected tiles of same type/owner using BFS
- `calculateBlocCosts(bloc, automated)`: Calculates Food/Energy/Labor/Ore costs to power a bloc
- `calculateProduction(bloc, board, powered, fedTileIds, automated)`: Calculates output for a bloc
  - Counts center dots
  - Counts connected partial dots between tiles
  - Applies automation multipliers
  - Returns ProductionInfo with amount and detailed logs
- `calculateGlobalProduction(board, player)`: Calculates maximum production across all blocs

**Production Algorithm**:
1. Find all connected tiles (bloc)
2. Calculate operating costs based on bloc size
3. Count production dots (centers + shared edges/corners)
4. Apply automation bonuses if applicable
5. Return commodity output

#### `placementLogic.ts`
**Purpose**: Validation logic for tile placement during Develop phase.

**Key Functions**:
- `isValidPlacement()`: Validates if a tile can be placed at a location
  - Checks for player's flag at location
  - Validates edge adjacency rules (half-dots must match)
  - Validates corner adjacency rules (third-dots must align)
  - Can be bypassed with `force` mode for testing
- `getValidPlacements()`: Returns all valid positions and orientations for a tile type

**Adjacency Rules**:
- Half-dots on edges must match commodity type with neighbors
- Third-dots on corners must match with 2 other adjacent tiles
- Prevents invalid tile orientations that would create mismatched connections

#### `marketDefinitions.ts`
**Purpose**: Market pricing structure.

**Key Exports**:
- `MARKET_STEPS`: Array of 12 price levels with buy/sell prices
  - Index 0 (top): Empty market, highest prices (sell: $40, buy: $44)
  - Index 11 (bottom): Full market, lowest prices (sell: $2, buy: $5)
  - Markets start at index 4 (middle) with 4 cubes

**Market Mechanics**:
- Selling: Place commodity in highest empty well, receive sell price
- Buying: Take commodity from lowest filled well, pay buy price
- Stock level determines current price index

### `/src/components/` Directory

#### `/components/game/`

**`Board.tsx`**
- Renders the hexagonal board as an SVG
- Props: board state, players, callbacks, highlighting
- Manages viewBox for auto-fit
- Delegates individual hex rendering to `Hex` component

**`Hex.tsx`**
- Renders a single hexagonal cell
- Shows flags, industry tiles, dots, features
- Handles click events for cell selection
- Renders ghost previews for tile placement
- Complex SVG rendering with rotation transforms for tile orientation

**`MarketBoard.tsx`**
- Displays all 5 commodity markets
- Shows current stock levels and prices
- Visual representation of market wells/price tracks

**`PlayerRoster.tsx`**
- Shows all players with their resources
- Displays current player indicator
- Shows resource counts and money

**`PlayerDashboard.tsx`**
- Detailed view of a single player's state
- Resource inventory, money, loans, flags remaining

**`ControlPanel.tsx`**
- Tool selection interface (tiles, flag, eraser, rotate, etc.)
- Phase information display
- Action buttons (Pass, Produce)

#### `/components/ui/`

**`ResourceIcon.tsx`**
- Reusable component for displaying commodity/resource icons
- Color-coded by commodity type

### `/src/pages/`

#### `Game.tsx`
**Purpose**: Main game page that orchestrates all components and game flow.

**Key Responsibilities**:
- Uses `useGameEngine` hook for game state
- Manages UI state (selected cell, tool, force mode, etc.)
- Handles production phase configuration:
  - Bloc power settings
  - Tile feeding selections
  - Automation toggle
- Implements interaction modes (idle, placing, moving)
- Calculates valid placements for selected tools
- Renders all game components with proper props
- Handles user interactions and delegates to game engine

**State Management**:
- Game state from `useGameEngine`
- Local UI state for selections and temporary data
- Production configuration state for bloc management
- Hover/highlight state for visual feedback

**Component Composition**:
```
Game
├── ControlPanel (tools, actions)
├── Board (hex grid, tiles, flags)
├── MarketBoard (all markets)
├── PlayerRoster (player list)
└── PlayerDashboard (current player details)
```

### `/src/tests/integration/` & `/e2e/`

**Testing Strategy**:
- **Unit Tests** (`npm run test`): Located alongside code or in `src/tests/`. Run with Vitest.
  - Utilities: `production.test.ts`, `placementLogic.test.ts`
  - Integration: `src/tests/integration/sharedDots.test.ts` (complex interactions)
  - Game Flow: `src/pages/Game.production.test.ts`
- **End-to-End Tests** (`npx playwright test`): Located in `/e2e/`. Run with Playwright.
  - Full browser verification of game scenarios
  - Multiplayer flows (`multiplayer-game.spec.ts`)
  - Save/Load functionality (`save-load-game.spec.ts`)

**Test Setup**:
- Vitest as test runner
- React Testing Library for component testing
- jsdom for DOM simulation

## Game Flow

### Phase Cycle

```
Trade Phase
  ├─ Players take turns buying/selling commodities
  ├─ Can take loans (promissory notes)
  └─ Pass to skip turn

Develop Phase
  ├─ Players take turns placing flags and building tiles
  ├─ Can rotate, move, or automate tiles
  └─ Pass to skip turn

Produce Phase
  ├─ Configure each bloc (power, feed, automate)
  ├─ Pay operating costs
  ├─ Receive commodity production
  └─ Single simultaneous phase (no turn order)
```

### State Flow

1. **User Interaction** → UI Component
2. **Event Handler** → Game.tsx
3. **Action Dispatch** → useGameEngine.handleAction()
4. **State Update** → GameState modified
5. **Re-render** → Components reflect new state

## Design Patterns

### Immutable State Updates
All state modifications use functional updates to ensure React detects changes:
```typescript
setGameState(prev => ({ ...prev, phase: nextPhase }))
```

### Coordinate System
Hexagonal coordinates stored as `{q, r}` objects, converted to `"q,r"` strings for object keys.

### Component Hierarchy
- **Smart Components**: `Game.tsx` - handles logic and state
- **Presentational Components**: Most components in `/components/` - receive props and render
- **Hook-based State**: `useGameEngine` encapsulates game logic

### Validation Pattern
Actions validated before state changes:
```typescript
if (!isValid) return { success: false, message: 'Invalid action' };
// Apply state change
return { success: true };
```

## Data Flow

### Board State
```
GameState.board: Record<string, HexCell>
├─ Key: "q,r" coordinate string
└─ Value: HexCell with occupant (Flag or Industry+Tile)
```

### Player Resources
```
Player.resources: Record<CommodityType, number>
Player.money: number
Player.loans: number (promissory notes)
```

### Market State
```
GameState.markets: Record<CommodityType, MarketState>
├─ stock: number (0-12, number of cubes in market)
└─ priceIndex: number (0-11, current price level)
```

## Key Algorithms

### Bloc Identification (BFS)
```
1. Start with a tile
2. Find all adjacent tiles of same type and owner
3. Use breadth-first search with visited set
4. Return connected component
```

### Production Calculation
```
1. Identify bloc
2. Calculate costs (Energy, Food, Labor, Ore based on size)
3. Count center dots (1 per tile)
4. Count shared edge dots (half + half = full)
5. Count shared corner dots (third + third + third = full)
6. Apply automation multiplier
7. Return total production
```

### Valid Placement Check
```
1. Verify cell has player's flag
2. For each edge with half-dot:
   - Check neighbor tile
   - Verify commodities match
3. For each corner with third-dot:
   - Check 2 neighbor tiles
   - Verify all commodities match
4. Return valid/invalid with reason
```

## Future Enhancements

Based on the rules.md and current implementation:

1. **Network Multiplayer**: Currently hotseat only
2. **Package Drafting**: Initial setup phase not fully implemented
3. **End Game Conditions**: Victory point calculation and game end
4. **Save/Load**: Persist game state

## Development Workflow

### Running the Project
```bash
npm run dev      # Start development server
npm run build    # Production build
npm run test     # Run tests
npm run lint     # Check code quality
```

### Adding New Features

1. **New Game Action**: Add case to `useGameEngine.handleAction()`
2. **New Component**: Create in appropriate `/components/` subdirectory
3. **New Game Logic**: Add utility function to `/utils/`
4. **New Type**: Extend types in `gameState.ts`

### Testing Philosophy
- Unit test utility functions
- Integration test complex game scenarios
- Test production rules against actual game rules

## File Responsibilities Summary

| File/Directory | Responsibility |
|---|---|
| `types/gameState.ts` | Type definitions |
| `hooks/useGameEngine.ts` | Game state management |
| `utils/hexUtils.ts` | Hex grid math |
| `utils/tileDefinitions.ts` | Tile specifications |
| `utils/production.ts` | Production calculations |
| `utils/placementLogic.ts` | Placement validation |
| `utils/marketDefinitions.ts` | Market pricing |
| `components/game/Board.tsx` | Board rendering |
| `components/game/Hex.tsx` | Individual hex rendering |
| `components/game/MarketBoard.tsx` | Market display |
| `components/game/PlayerRoster.tsx` | Player list |
| `components/game/ControlPanel.tsx` | Tool selection |
| `pages/Game.tsx` | Main game orchestration |
| `App.tsx` | Application entry |

## Notes for AI Agents

When working on this codebase:

1. **State is Immutable**: Always use spread operators for state updates
2. **Coordinate Strings**: Board keys are `"q,r"` format, parse with `stringToCoords()`
3. **Type Safety**: Leverage TypeScript; avoid `any` types
4. **Game Rules**: Reference `resources/rules.md` for authoritative game mechanics
5. **Testing**: Add tests when modifying game logic in `/utils/`
6. **Component Props**: Most components are presentational; logic stays in `Game.tsx` or `useGameEngine`
7. **SVG Rendering**: Hex and tile rendering uses complex SVG transforms; be careful with rotations
8. **Phase-Specific Logic**: Many actions are only valid in specific game phases

## Common Tasks

### Adding a New Tile Type
1. Add to `IndustryType` in `gameState.ts`
2. Define in `TILE_DEFINITIONS` in `tileDefinitions.ts`
3. Add rendering logic in `Hex.tsx` if needed
4. Add to tool selection in `ControlPanel.tsx`

### Modifying Production Rules
1. Update logic in `production.ts`
2. Add/update tests in `production.test.ts`
3. Verify against `resources/rules.md`

### Adding UI Features
1. Create component in `/components/ui/` or `/components/game/`
2. Import and compose in `Game.tsx`
3. Add necessary state to `Game.tsx` or `useGameEngine.ts`

---
> Source: [mermerico/WealthOfNations](https://github.com/mermerico/WealthOfNations) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
