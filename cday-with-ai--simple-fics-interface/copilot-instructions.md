## simple-fics-interface

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Simple-FICS-Interface** - A modern chess client for the Free Internet Chess Server (FICS) built as a React/TypeScript monorepo.

- **Current Branch**: `main` (main development branch)
- **Architecture**: Monorepo using Yarn workspaces
- **Deployment**: Cloudflare Pages (simple-fics-interface.pages.dev)
- **Main Packages**:
  - `@fics/shared`: Core business logic, chess engine, FICS protocol
  - `@fics/web`: React web application (Vite)
  - `@fics/mobile`: React Native mobile app (in development)

## Quick Start

```bash
# Install dependencies
yarn install

# Run web development server (default port 5173)
yarn web

# Run web development server in dev mode (port 14040)
yarn dev

# Run web development server for Claude (port 14080)
yarn claude

# Build web app
yarn build:web
```

The app can also be opened directly via `index.html` in the root directory, which loads the pre-built app from `packages/web/dist/`.

## Architecture & Key Components

### Core Services (packages/shared/src/services/)

- **ChessAPI**: Custom chess engine supporting multiple variants
  - Standard, Chess960, Crazyhouse, Suicide, Atomic, Losers, Wild
  - Full move validation and generation
  - ~98% test coverage with 400+ tests
  - Key files:
    - `ChessAPI.ts`: Main API interface
    - `ChessAPI.execution.ts`: Move execution logic
    - `ChessAPI.moveGeneration.ts`: Legal move generation
    - `ChessAPI.validation.ts`: Move validation
    - `ChessAPI.variants.ts`: Variant-specific rules

- **FICSProtocol**: WebSocket-based FICS communication
  - Timeseal2 protocol implementation
  - Style12 board update parsing
  - Comprehensive message parsing
  - Security hardened against XSS

- **StockfishEngine**: WebAssembly integration
  - Stockfish 16 WASM
  - Requires CORS headers (handled by `_headers` file)

### State Management (packages/shared/src/models/)

MobX stores for reactive state:
- `GameStore`: Chess game state, move history, clocks
- `FICSStore`: Server connection, protocol handling
- `ChatStore`: Tab-based chat system
- `PreferencesStore`: User settings
- `AnalysisStore`: Stockfish analysis state

### Web Application (packages/web/)

- **Tech Stack**: React 18, TypeScript, Vite, styled-components
- **Key Components**:
  - `ChessGameLayout`: Main game interface
  - `ChessBoardWithPieces`: Board rendering and interaction
  - `ChatPanel`: Multi-tab chat interface
  - `PlayerCard` & `ObservableClock`: Player info display
  - `GameControls`: Draw/resign/analysis buttons

## Important Technical Notes

### 1. WebAssembly Requirements
Stockfish requires these headers (set in `_headers` for Cloudflare):
```
Cross-Origin-Embedder-Policy: require-corp
Cross-Origin-Opener-Policy: same-origin
```

### 2. Build & Deployment
- App builds to `packages/web/dist/`
- Built files are committed to repository
- Root `index.html` loads the built app
- Cloudflare Pages serves directly without build step

### 3. Chess Move Handling
```typescript
// Moves can be made via:
gameStore.makeMove(from, to, promotion); // Long algebraic
gameStore.makeSANMove("Nf3"); // Standard algebraic

// For FICS, moves are sent as:
ficsStore.sendCommand("e2e4"); // Long algebraic format
```

### 4. Testing
```bash
# Run all tests
yarn test

# Run specific package tests
yarn workspace @fics/shared test

# Watch mode
yarn workspace @fics/shared test:watch
```

### 5. Common Development Tasks

#### Adding a New Chess Variant
1. Add variant type to `ChessAPI.types.ts`
2. Implement rules in `ChessAPI.variants.ts`
3. Add comprehensive tests in `__tests__/`
4. Update variant mapping in `FICSStore`

#### Working with FICS Protocol
1. New message types go in `FicsProtocol.types.ts`
2. Parser logic in `FicsProtocol.parser.ts`
3. Handle in `FICSStore.processMessages()`
4. All user input must be sanitized

#### UI Component Guidelines
- Use styled-components with transient props (`$propName`)
- Follow existing component patterns
- Isolate frequently updating components (e.g., clocks)
- Use MobX `observer` for reactive components

### 6. Performance Considerations

- Clock updates run at 100ms intervals
- Piece animations disabled when < 10 seconds
- Use `React.memo` and `observer` strategically
- Board renders are optimized with position diffing

### 7. Current State Notes

- All old vanilla JS code has been removed
- Assets organized in `packages/web/public/`
- No duplicate files at root level
- Player names are clickable (not right-click) for context menu
- Console tab shows FICS output with smart scrolling

## Development Modes

The project supports multiple development server configurations:

- **`yarn web`** - Standard development server (port 5173)
  - Default Vite configuration
  - Auto-opens browser
  - Hot module replacement enabled

- **`yarn dev`** - Enhanced development mode (port 14040)
  - Development-specific optimizations
  - External network access enabled
  - Ideal for team development

- **`yarn claude`** - Claude-specific development mode (port 14080)
  - Optimized for Claude Code assistance
  - External host access enabled
  - Separate port to avoid conflicts

## Common Commands Reference

```bash
# Development
yarn web                    # Start dev server (port 5173)
yarn dev                    # Enhanced dev mode (port 14040)
yarn claude                 # Claude dev mode (port 14080)
yarn build:web             # Build for production
yarn test                  # Run all tests
yarn typecheck             # Check TypeScript

# FICS Commands (via UI)
tell <user> <message>      # Private message
<number>. <message>        # Channel message
observe <game>             # Watch a game
seek <time> <inc>          # Seek a game
```

## Debugging Tips

1. **Chess moves not working**: Check browser console for validation errors
2. **Stockfish not loading**: Verify CORS headers in Network tab
3. **Board not updating**: Check Style12 parsing in console logs
4. **Chat issues**: Verify WebSocket connection status

## Code Style

- TypeScript strict mode enabled
- ESLint configuration in place
- Prettier for formatting
- No console.log in production code (except error handling)
- Comprehensive error boundaries

Remember: This is a production app deployed on Cloudflare Pages. All changes should maintain backwards compatibility and consider performance implications.

---
> Source: [cday-with-ai/Simple-FICS-Interface](https://github.com/cday-with-ai/Simple-FICS-Interface) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
