## chatagotchi

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is "Chatagotchi" - a virtual pet game built as an MCP (Model Context Protocol) server with OpenAI Apps SDK widgets. The game runs inside ChatGPT where users raise virtual pets through feeding and playing actions, with an achievement system tracking their progress.

## Key Commands

### Frontend (Widgets)
- `npm run build` - Build all widgets to `assets/` directory (generates separate CSS/JS files per widget)
- `npm run dev` - Start Vite dev server on port 4444
- `npm run serve` - Serve built assets with CORS enabled

### Backend (MCP Server)
- `cd mcp_server_node && npm start` - Start the MCP server (runs with tsx)
- Server listens on port configured in `mcp_server_node/src/config.ts`

## Architecture

### Three-Layer System

**1. MCP Server (`mcp_server_node/src/`)**
- `server.ts` - MCP tool definitions (thin controllers)
- `game-service.ts` - Business logic layer encapsulating game state management
- `game-logic.ts` - Pure functions for game mechanics (stat changes, lifecycle, achievements)
- `stytch.ts` - User authentication and metadata persistence
- `index.ts` - Express server with Stytch OAuth + MCP endpoint

**2. Shared Types (`shared-types/`)**
- `game-types.ts` - Single source of truth for types used by both backend and frontend
- Exports: `PetState`, `PetSpecies`, `PetLifecycleState`, `Achievement`, `AchievementState`, `ACHIEVEMENTS`, `SPECIES_EMOJIS`
- Imported by both MCP server and widget code

**3. Frontend Widgets (`widgets/`)**
- `pet/` - Main pet display widget (shows stats, lifecycle stage, action buttons)
- `achievements/` - Achievement gallery widget (shows locked/unlocked achievements)
- `utils/` - Shared React hooks (`useOpenAiGlobal`, `useWidgetState`, etc.)

### Data Flow

1. **ChatGPT** calls MCP tool (e.g., `pet-feed`)
2. **server.ts** instantiates `GameService` with user auth
3. **GameService** reads game state from Stytch metadata, applies action via `game-logic.ts`, checks achievements, saves state
4. **GameService** returns `{ petState, newAchievements, message }` to server
5. **server.ts** returns response with `structuredContent` and `_meta['openai/outputTemplate']` pointing to widget
6. **ChatGPT** renders widget HTML, hydrates with `structuredContent` via Skybridge

### Game State Storage

- **Pet state** - Stored per-game in `userMetadata.petState` (wiped on new game)
- **Achievement state** - Persistent across games in `userMetadata.achievementState`
- Both managed together by `GameService.getGameState()` / `saveGameState()`

### Game Mechanics

- Pet has 3 stats: stamina, happiness, health (0-100 scale, die if any drops below 20)
- 4 food options, 3 play activities (each affects stats differently)
- Lifecycle: BABY → CHILD → ADULT → COMPLETE (based on turn count)
- 11 discovery achievements: 5 species completions, 6 death types (including secret deaths like skiing crash and overeating)

### Widget System

- Each widget has `index.html`, `index.tsx`, `index.css`, `types.ts`
- Built with Vite into separate bundles (configured in `vite.config.mts`)
- Widgets use `useOpenAiGlobal('toolOutput')` hook to access `structuredContent` from MCP response
- Widget entry points must be registered in `vite.config.mts` rollupOptions.input

### MCP Tool Structure

Tools return:
```typescript
{
  content: [{ type: 'text', text: 'User-facing message' }],
  structuredContent: { petState, lastAction, ... }, // Data for widget
  _meta: { 'openai/outputTemplate': 'ui://widget/pet.html' }
}
```

## Important Patterns

### Adding New Types
- Define in `shared-types/game-types.ts`
- Import in both `mcp_server_node/src/game-logic.ts` and widget type files
- Never duplicate type definitions

### Adding New Widgets
1. Create `widgets/your-widget/` with index.html, index.tsx, index.css, types.ts
2. Add entry to `vite.config.mts` rollupOptions.input
3. Register resource in `server.ts` with `server.registerResource()`
4. Add tool that returns `_meta['openai/outputTemplate']` pointing to your widget URI

### GameService Pattern
- Always instantiate with `authInfo` in tool handlers: `new GameService(authInfo)`
- Use public methods: `startNewGame()`, `feedPet()`, `playWithPet()`, `getAchievements()`
- GameService handles ALL state persistence - don't call Stytch directly from server.ts

### Authentication
- MCP server requires Stytch OAuth bearer tokens
- `STYTCH_DOMAIN`, `STYTCH_PROJECT_ID`, `STYTCH_SECRET` must be set in env
- User ID extracted from `authInfo.extra.subject`

## Deployment

Frontend widgets are hosted on Vercel (configured in `vercel.json`). The MCP server references widget URLs via `https://chatagotchi-jet.vercel.app/{widget}.{css|js}` in resource definitions.

For local testing, use ngrok to expose MCP server and add to ChatGPT Settings > Connectors.

---
> Source: [stytchauth/chatagotchi](https://github.com/stytchauth/chatagotchi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
