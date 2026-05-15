## quiet-hex-logic

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Hexology is a cross-platform board game platform built with React/Vite for web and Expo for mobile. It supports 5 games (Hex, Chess, Checkers, Tic Tac Toe, Connect 4) via a unified game engine registry. Features include multiplayer lobbies, Elo-rated matchmaking, AI opponents, tournaments, puzzles, Discord Activity integration, and a mod system.

This is a Lovable (lovable.dev) project. The `lovable-tagger` dev dependency adds component tagging in development mode only.

## Development Commands

```bash
# Web
npm run dev              # Vite dev server on port 8080
npm run build            # Production build
npm run lint             # ESLint

# Tests (Vitest)
npm test             # Run all tests (92 tests across 5 engines)
npm test:watch       # Watch mode
npx vitest run src/lib/hex  # Run tests for a single game engine

# Mobile (Expo)
npm run ios              # iOS
npm run android          # Android
```

Test files live at `src/lib/<game>/__tests__/engine.test.ts`. Vitest is configured with the `@/*` path alias.

## Path Alias

`@/*` maps to `./src/*` (configured in tsconfig.json, vite.config.ts, and vitest.config.ts). Always use it for imports.

## Architecture

### Frontend Stack

- **React 18** + TypeScript, Vite, react-router-dom
- **Tailwind CSS** + shadcn/ui components (`src/components/ui/`)
- **TanStack React Query** for server state
- **Supabase** for auth, database, and realtime subscriptions
- No global state library — React Query + custom hooks + local state

### TypeScript Configuration

Strictness is relaxed: `noImplicitAny: false`, `strictNullChecks: false`, `noUnusedLocals: false`. Be cautious with null/undefined at runtime.

### Routing (`src/App.tsx`)

Provider wrap order: `DiscordProvider` → `QueryClientProvider` → `TooltipProvider` → `DiscordActivityWrapper` → `BrowserRouter`.

Key routes: `/` (home), `/auth`, `/lobby/:lobbyId`, `/match/:matchId`, `/tournament/:tournamentId`, `/replay/:matchId`, `/profile/:userId`, `/puzzles`, `/premium`, `/leaderboard`, `/mods`.

## Game Engine Registry

The central architectural pattern. All games are registered in `src/lib/engine/registry.ts` and accessed through a common `GameEngine<TMove>` interface (`src/lib/engine/types.ts`).

### Interface

```typescript
interface GameEngine<TMove> {
  currentPlayer(): 1 | 2;
  ply(): number;
  isLegal(move: TMove): boolean;
  applyMove(move: TMove): void;
  winner(): 0 | 1 | 2;
  isDraw(): boolean;
  isGameOver(): boolean;
  clone(): GameEngine<TMove>;
  serializeMove(move: TMove): Record<string, unknown>;
  deserializeMove(data: Record<string, unknown>): TMove;
}
```

### Registry API

- `registerGame(def)` — Register a game definition
- `getGame(key)` — Lookup by key (throws if not found)
- `listGames()` — All registered definitions
- `createEngine(key, opts)` — Factory

### Per-Game Structure

Each game follows this layout:
- `src/lib/<game>/engine.ts` — Core engine class
- `src/lib/<game>/ai.ts` — AI implementation
- `src/lib/<game>/__tests__/engine.test.ts` — Engine tests
- `src/lib/engine/adapters/<game>Adapter.ts` — Wraps engine behind `GameEngine<TMove>`
- `src/components/<game>/<Game>Board.tsx` — React board component
- `supabase/functions/_shared/validators/<game>.ts` — Server-side validator

### Move Types

| Game | TMove | Serialized Key |
|------|-------|----------------|
| Hex | `number \| null` | `{ cell }` |
| Chess | `{ uci: string; promotion?: string }` | `{ uci, promotion }` |
| Checkers | `{ path: number[] }` | `{ path }` |
| TTT | `number` | `{ cell }` |
| Connect4 | `number` | `{ col }` |

### Adding a New Game

Only requires: engine + adapter + board component + `registerGame()` call + server validator + add to `createValidator()` switch in `apply-move`. The registry drives CreateLobby, MatchBoard, useMatchState, and useMatchActions automatically. Wire per-game last-move state into `useMatchState.ts`, `useMatchActions.ts`, `MatchBoard.tsx`, and `Match.tsx`.

## Critical Pattern: Move Replay

The database stores **moves**, not board state. Every match load reconstructs the engine by replaying the full move history:

```typescript
const adapter = getGame(gameKey).createEngine(opts);
for (const m of moves) {
  const move = adapter.deserializeMove(m.move);
  adapter.applyMove(move);
}
```

Never assume board state is stored directly. This is implemented generically in `useMatchState.ts`'s `replayMoves()`.

## Key Hooks

- `useMatchState(matchId)` — Loads match + moves, replays engine state via registry, manages per-game last-move state. Handles online, local, and Discord local matches.
- `useMatchActions(...)` — Per-game move handlers (`handleCellClick`, `handleChessMove`, `handleTttMove`, `handleCheckersMove`, `handleConnect4Move`). Each validates locally, applies optimistically, then submits to `apply-move` edge function.
- `useAIOpponent(...)` — AI move execution (currently Hex-focused for Discord local matches; other games use `createAI` from registry).

## Hex Game Engine (`src/lib/hex/engine.ts`)

The most complex engine. Uses Disjoint Set Union (DSU) for O(1) amortized win detection:

- **Coordinates**: Odd-q offset hex grid. Neighbor calculations differ for even vs odd columns.
- **Players**: Player 1 (indigo) connects West↔East. Player 2 (ochre) connects North↔South.
- **Pie rule**: On ply 1, Player 2 can swap all colors. Rebuilds entire DSU after swap.
- **Winning path**: `getWinningPath()` returns ALL cells connected to both borders, not the shortest path.

## Supabase Integration

### Auto-Generated Types

`src/integrations/supabase/types.ts` is auto-generated from the database schema. Do not edit manually.

### Key Tables

- `matches` — Core game state with `version` field for optimistic concurrency and `game_key` for game type
- `moves` — Move history with `action_id` (UUID) for idempotency. Move data in JSONB `move` column.
- `match_players` — Player roles (color, is_bot, rating_change)
- `lobbies` / `lobby_players` — Pre-game rooms with settings
- `profiles` — User data (elo_rating, is_premium, discord_id, is_guest)

### Realtime Pattern

Canonical example: `src/hooks/useLobby.ts`:
1. Subscribe to `postgres_changes` on relevant tables with filters
2. Handle payload in callback (update local state or refetch)
3. Clean up with `supabase.removeChannel(channel)` in useEffect return
4. 15-second heartbeat interval updates `last_seen` for presence

### Concurrency & Idempotency

- **Optimistic concurrency**: Match updates use `WHERE version = old_version`. Failed updates (409) require client retry.
- **Idempotent moves**: Client generates UUID `action_id` before submission. Edge function deduplicates.
- **Rate limiting**: `check_move_rate_limit` RPC prevents move spam.

## Edge Functions (`supabase/functions/`)

All Deno edge functions follow this pattern:
1. CORS preflight handling
2. Auth via `supabase.auth.getUser(token)` from Authorization header
3. Input validation with Zod schemas
4. Business logic + database operations
5. JSON response

The `apply-move` function uses a `createValidator(gameKey)` factory to dispatch to game-specific `ServerValidator` implementations in `supabase/functions/_shared/validators/`. Each validator implements `replayMove()` and `applyProposedMove()`.

Move flow: Client → `apply-move` (validates + applies + increments match version) → realtime subscription notifies opponent.

## Discord Activity Integration

Discord Activities can't make arbitrary HTTP requests. All backend calls route through a proxy: `/api` → Supabase Functions URL. The `DiscordContext` (`src/lib/discord/DiscordContext.tsx`) handles SDK init, OAuth token exchange, and participant tracking. Local matches use IDs prefixed with `discord-`.

## Styling

- Custom theme colors: `paper`, `ink`, `indigo`, `ochre`, `graphite` (CSS variables)
- Fonts: Spectral (body), IBM Plex Mono (mono)
- Dark mode is the default

---
> Source: [Zwin-ux/quiet-hex-logic](https://github.com/Zwin-ux/quiet-hex-logic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
