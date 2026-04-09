## aetherrpg

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

```bash
# Development
npm run dev          # Start dev server (client + server with hot reload)
npm run check        # Run TypeScript type checking

# Database
npm run db:push      # Push schema changes to Neon PostgreSQL database

# Production
npm run build        # Build client (Vite) + server (esbuild)
npm run start        # Start production server (requires build first)
```

## Architecture Overview

**Monorepo Structure**: Single repository with client (React SPA) and server (Express API) sharing TypeScript types.

- `client/` - React 18 + TypeScript + Vite frontend
- `server/` - Express + TypeScript API server
- `shared/` - Shared types and database schema (Drizzle ORM)

### Client Architecture

**State Management**: Zustand stores with `subscribeWithSelector` middleware. Each store manages a distinct domain:

- `useCharacter` - Character stats, level, class, equipment
- `useInventory` - Items, equipment slots, consumables
- `useStoryEngine` - Story nodes, choices, narrative flow
- `useAIAgents` - Template-based AI responses (DM, NPCs)
- `useOracle` - Dice oracle for situation outcomes
- `useAudio` - Background music and sound effects
- `useQuests` - Quest tracking and objectives
- `useAchievements` - Achievement unlocks and progress

Stores live in `client/src/lib/stores/` and are consumed by components via hooks.

**Story System**: Dual-mode story engine:
1. **CustomStoryEngine** (`client/src/lib/customStoryEngine.ts`) - Hardcoded story graph with nodes and choices
2. **Ink.js integration** - Can load `.ink.json` files for more complex branching narratives

Story state (current node, variables, choices) persists to localStorage via `saveProgress()` / `loadProgress()`.

**AI Agents**: Template-based responses (no external API). Agents are defined in `client/src/lib/aiAgents.ts` with personality traits and response templates keyed by context (e.g., `combat_start`, `victory`, `greeting`). The `AIAgentEngine` picks random responses and personalizes them based on player name and agent role.

**Oracle Engine**: Five-die poker oracle (`client/src/lib/oracleEngine.ts`) maps dice rolls to tiers (0-5) and generates narrative outcomes from hand-crafted templates. Used for resolving player-described situations.

### Server Architecture

**API Routes**: Defined in `server/routes.ts`. RESTful endpoints for:
- `/api/auth/*` - Register, login, logout (bcrypt password hashing, session-based)
- `/api/characters/*` - CRUD operations for characters (tied to user ID)
- `/api/saves/*` - Game state save/load by slot
- `/api/leaderboard/*` - Get/submit leaderboard entries
- `/api/analytics/event` - Track user events (optional)

**Database**: PostgreSQL (Neon serverless) via Drizzle ORM. Schema in `shared/schema.ts` defines:
- `users` - Authentication (username, hashed password)
- `characters` - Character data as JSONB
- `gameSaves` - Game state snapshots by slot
- `leaderboardEntries` - Player rankings
- `analyticsEvents` - Event tracking

**Sessions**: Express sessions stored in-memory (memorystore). Production should use `connect-pg-simple` for persistent session storage.

### Data Layer

Static game data (items, spells, quests, etc.) lives in `client/src/data/*.ts` as TypeScript objects. Examples:
- `items.ts` - Weapons, armor, consumables with stats
- `spells.ts` - Magic spells with effects and mana costs
- `quests.ts` - Quest definitions with objectives
- `talents.ts` - Skill tree nodes
- `achievements.ts` - Achievement definitions

These are imported directly into components/stores as needed.

### UI Components

Built on **Radix UI** primitives + **Tailwind CSS**. Component library in `client/src/components/ui/` provides:
- `button`, `card`, `dialog`, `tabs`, `textarea`, etc.

Game-specific components in `client/src/components/game/`:
- `GameInterface.tsx` - Main game container with tabbed layout
- `CharacterCreation.tsx` - Class selection and stat allocation
- `StoryDisplay.tsx` - Renders current story text
- `ChoiceButtons.tsx` - Interactive choice selection
- `InventoryPanel.tsx` - Item management
- `OraclePanel.tsx` - Dice oracle consultation UI
- `CharacterSheet.tsx` - Full character stats display

## Key Patterns and Conventions

### Adding New Zustand Stores

1. Create `client/src/lib/stores/useYourFeature.tsx`
2. Use `subscribeWithSelector` middleware for reactive subscriptions
3. Define state interface with actions (e.g., `initialize`, `update`, `reset`)
4. Export typed hook: `export const useYourFeature = create<YourState>()(subscribeWithSelector(...))`

### Adding Story Nodes (CustomStoryEngine)

Story nodes are defined in `createStoryData()` inside `customStoryEngine.ts`. Each node has:
```typescript
{
  id: 'unique_id',
  text: 'Story text with markdown support',
  choices: [
    { text: 'Choice text', destination: 'target_node_id' }
  ],
  tags: ['location:town', 'character:npc_name', 'item:sword']
}
```

Tags are used to trigger effects (item acquisition, quest updates, etc.).

### Adding AI Agent Responses

Edit `client/src/lib/aiAgents.ts`:
1. Add new agent to `agents` array with `id`, `role`, `personality`, `knowledge`
2. Define `responseTemplates` object keyed by context
3. Use `getResponse(agentId, context, {playerName})` to generate responses

### Database Schema Changes

1. Edit `shared/schema.ts` (Drizzle schema)
2. Run `npm run db:push` to sync with Neon database
3. Update TypeScript types (auto-inferred via `typeof table.$inferSelect`)

### Environment Variables

Copy `.env.example` to `.env` and configure:
- `DATABASE_URL` - PostgreSQL connection string (required for db operations)
- `SESSION_SECRET` - Secret for session encryption
- `VITE_*` - Client-side feature flags (multiplayer, cloud saves, leaderboards)

All `VITE_*` vars are injected into client bundle and accessible via `import.meta.env`.

## Important Architectural Details

**Game State Persistence**: Two-tier system:
1. **Client-side** - localStorage for quick save/load (no network required)
2. **Server-side** - PostgreSQL for cloud saves (requires authentication)

Both use the same `GameState` type defined in `client/src/types/game.ts`.

**Character Progression**: XP thresholds defined in character store. Leveling up triggers stat point allocation and potential talent unlocks. Stats affect combat calculations (damage, armor, hit chance).

**Combat System**: Turn-based with enemy AI. Damage formula incorporates character stats, weapon stats, and random variance. Status effects (poison, buffs, etc.) apply per-turn modifiers.

**Item Rarity System**: `common | uncommon | rare | epic | legendary` affects stat bonuses and visual presentation (color coding in UI).

**Quest System**: Each quest has objectives with completion tracking. Objectives can be `kill`, `collect`, `reach_location`, `talk_to_npc`, or `use_item`. Quest rewards grant XP, gold, and items.

**Ink.js Integration**: If using Ink instead of CustomStoryEngine, place `.ink.json` files in `client/src/data/` and load via `inkStory.ts` utilities. Ink provides richer conditional logic and variable tracking than the custom engine.

## Testing and Debugging

**Type Checking**: Run `npm run check` before committing. The codebase uses strict TypeScript.

**Hot Reload**: Vite dev server watches `client/src/**` for changes. Server restarts on `server/**` changes via `tsx` watch mode.

**Database Debugging**: Drizzle logs SQL queries in development mode. Check server console for query output.

**Story Testing**: Use browser DevTools → Application → Local Storage to inspect saved story state. Key: `rpg_story_save`.

**Session Debugging**: Sessions stored in-memory by default. To inspect: add breakpoints in `server/routes.ts` and check `req.session.userId`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Cybersoulja)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Cybersoulja)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
