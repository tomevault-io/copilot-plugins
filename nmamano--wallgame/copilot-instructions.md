## wallgame

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

See `.cursor/rules/` for general agent rules and `/info/` for detailed design documentation.

## Project Overview

Wallgame is a real-time multiplayer strategy board game with:

- Local play, online multiplayer with matchmaking, spectator mode, game replay
- Rankings, puzzle challenges, and solo campaigns
- Advanced AI powered by MCTS + neural networks (Deep-Wallwars engine)
- Custom bot integration via WebSocket protocol

## Monorepo Structure

```
wallgame/
├── frontend/                 # React 19 web application
├── server/                   # Hono backend (API + WebSocket)
├── shared/                   # Shared TypeScript (contracts + domain logic)
├── deep-wallwars/            # C++ AI engine + Python training pipeline
├── dummy-engine/             # Reference bot implementation
├── official-custom-bot-client/  # Bot integration client
├── tests/                    # Integration tests
├── drizzle/                  # Database migrations (SQL)
├── scripts/                  # Build & migration scripts
├── assets/                   # Game assets, fonts, models
└── info/                     # Design documentation (26 .md files)
```

## Commands

```bash
# Development
bun run dev                    # Start backend server (port 3000)
cd frontend && bun run dev     # Start Vite frontend (port 5173)

# Build & Deploy
bun run build                  # Build frontend (tsc + vite)
bun run migrate                # Run Drizzle database migrations

# Code Quality
bun run format                 # Format with Prettier
bun run lint                   # Lint with ESLint (auto-fix)

# Testing & CI
bun run test                   # Run tests (uses WSL)
bun run ci                     # Full CI: format check, lint, test, build
```

For Deep-Wallwars build and training instructions, see `info/deep-wallwars-integration.md`.

**Development ports:** Frontend on 5173 proxies API/WebSocket to backend on 3000. Always use port 5173 for development.

**Runtime logs:** If the frontend, server, or bot client are running, all their outputs will be available in the `logs/` folder. If you can't find the logs there, ask the user to reset the scripts with logging enabled.

## Subprojects

### Frontend (`frontend/`)

React single-page application for the game UI.

**Tech:** React 19, Vite, TanStack Router, TanStack Query, Tailwind CSS v4, Radix UI, Lucide icons

```
frontend/src/
├── routes/          # TanStack Router file-based pages
├── components/      # React UI components (26 directories)
├── hooks/           # Game orchestration hooks
├── lib/             # Controllers, game client, utilities
├── game/            # Local game state logic
├── types/           # TypeScript interfaces
└── assets/          # Images, icons
```

**Key files:**

- `src/main.tsx` - Application entry point
- `src/hooks/use-game-page-controller.ts` - Main game orchestrator
- `src/hooks/use-online-game-session.ts` - WebSocket handshake
- `src/lib/controllers/` - Player controller implementations

### Server (`server/`)

Hono-based backend handling HTTP API, WebSocket connections, and game sessions.

**Tech:** Hono on Bun, PostgreSQL with Drizzle ORM, Kinde OAuth

```
server/
├── index.ts         # Main entry point (Hono setup)
├── kinde.ts         # OAuth authentication
├── routes/          # Hono API route handlers
├── db/              # Database layer
│   └── schema/      # Drizzle schema definitions
├── games/           # Game session management
│   ├── rating-system.ts
│   ├── persistence.ts
│   └── custom-bot-store.ts
└── chat/            # In-game chat features
```

### Shared (`shared/`)

TypeScript code shared between frontend and server.

```
shared/
├── contracts/       # API types with Zod validation
│   ├── websocket-messages.ts   # Game state updates, move commands
│   ├── custom-bot-protocol.ts  # Engine integration API
│   ├── games.ts, user.ts, ranking.ts, etc.
└── domain/          # Game rules & logic
    ├── game-state.ts           # Board state representation
    ├── game-utils.ts           # Move validation
    ├── classic-setup.ts        # Game variant configs
    └── dummy-ai.ts             # Simple AI implementations
```

### Deep-Wallwars (`deep-wallwars/`)

Advanced AI engine using Monte Carlo Tree Search with neural network evaluation.

**Tech:** C++17, CMake, CUDA, TensorRT, Python (PyTorch + FastAI for training)

```
deep-wallwars/
├── src/             # C++ source files (25+ files)
│   ├── game/        # Game logic
│   ├── mcts/        # Monte Carlo Tree Search
│   └── nn/          # Neural network integration
├── scripts/         # Python training scripts
│   └── training.py  # Main training orchestrator
├── models_*/        # Trained models (.pt, .onnx, .trt)
└── data_*/          # Self-play game data (CSV)
```

**Training loop:** Self-play (C++ MCTS) → CSV data → PyTorch training → TensorRT export

### Dummy Engine (`dummy-engine/`)

Reference bot implementation demonstrating the bot protocol.

**Tech:** Pure TypeScript/Bun (no dependencies)

Compiles to standalone executable. Simple AI that walks toward goal.

### Official Custom Bot Client (`official-custom-bot-client/`)

WebSocket client for connecting external bot engines to the game server.

**Tech:** TypeScript, Zod

```
official-custom-bot-client/src/
├── index.ts         # Main entry point
├── ws-client.ts     # WebSocket protocol
├── engine-runner.ts # External engine process management
└── dumb-bot.ts      # Fallback implementation
```

### Tests (`tests/`)

Integration and game logic tests.

```
tests/
├── integration/     # Integration test suites
└── game/            # Game logic tests
```

Run with `bun run test` (uses WSL).

### Database (`drizzle/`)

Drizzle ORM migrations. 15+ SQL migration files.

Schema defined in `server/db/schema/` (users, games, ratings, puzzles, etc.)

## Architecture

### Controller-Centric Game Flow

The core pattern abstracts transport differences between play modes:

- **GamePlayerController interface** - Unified API for all seat interactions
  - `LocalHumanController`: UI-driven local players
  - `RemotePlayerController`: Online players via WebSocket
  - `EasyBotController`: AI opponents
  - `SpectatorController`: Read-only viewing

- **Seat registries:**
  - `seatActionsRef`: Maps PlayerId → Controller (controllable seats)
  - `seatViewsRef`: Seat metadata (name, avatar, connection status)

### Data Flow

```
Frontend (React)
    ↓ WebSocket / HTTP
Server (Hono)
    ├→ PostgreSQL (Drizzle ORM)
    ├→ Bot Client → Deep-Wallwars Engine (AI)
    └→ Shared Domain (game rules)
```

### Key Design Decisions

1. **Transport-agnostic UI:** Controllers hide whether moves are local or networked
2. **Server-authoritative:** Online games wait for server confirmation before applying moves
3. **Spectator-first:** Spectator path is a first-class flow, not an error fallback
4. **History as pure snapshot:** `buildHistoryState()` creates immutable snapshots; `historyCursor` (null = live, number = ply index)
5. **Seat credentials:** Server mints ephemeral `{token, socketToken}` pairs per seat

## Code Style

- Use Bun, not npm
- Architecture-first: no backwards compatibility hacks needed
- Type-driven: prefer required fields over optionals
- Explicit over implicit: avoid race conditions with clear state transitions
- All frontend-server boundary types go in `shared/contracts/` with Zod schemas

## Implementation Quality Rules

When writing new code (features, components, layouts), follow these rules to avoid introducing
technical debt that requires immediate cleanup:

1. **Measure, don't predict.** Never hardcode pixel/rem constants to guess the rendered size of a
   component you don't own. If layout depends on another element's dimensions, use CSS flex/grid to
   let the browser compute it, or use ResizeObserver to measure it. Magic pixel budgets like
   `viewportHeight - 32 - 40 - 36 - ...` become silently wrong when any component changes.

2. **Import, don't redefine.** If a type or interface already exists in the codebase, import it.
   Never copy-paste a type definition into a new file — even a "simplified" version. Simplified
   copies with fewer fields still accept the full type (structural typing), so there's no reason
   to fork. Use the Explore agent or grep to check before defining a new interface.

3. **Use the type system to express intent.** If a prop is unused in some mode, make it optional
   (`prop?: Type`) rather than passing a sentinel value (`prop={0}`, `prop=""`, `prop={null}`).
   Sentinels lie to the reader — they look like real values. Optional props say "not applicable
   here" explicitly.

4. **Don't fork the render tree if you can branch within it.** Before creating a parallel JSX tree
   with an early return (like `if (mobile) return <MobileLayout/>; return <DesktopLayout/>`),
   check if the layouts share structure. Shared elements (matching panel, spectator banners,
   loading states) should be rendered once, with only the divergent portions gated by conditionals.
   Duplicated trees mean duplicated bugs.

## Fix Quality Rules

When fixing a bug, **always pause before implementing** and ask: "Am I patching a symptom, or
addressing the root cause?" Follow these rules strictly:

1. **Root-cause over symptom-patch.** If a fix requires adding multiple refs, flags, or guards to
   work around a design assumption, stop. The assumption is probably what needs changing. A correct
   fix should feel proportional to the bug — a one-line structural change beats a multi-effect
   state-machine patch every time.

2. **Present the approach before writing code.** For any non-trivial bug fix, describe the planned
   approach to the user and get approval BEFORE implementing. Include: (a) why the bug happens,
   (b) where the fix goes, (c) why this approach is clean vs alternatives. If you catch yourself
   adding complexity, that's a signal to pause and reconsider.

3. **Never stack hacks.** If a first fix attempt doesn't work, do NOT add more complexity on top.
   Step back, re-examine the root cause, and consider whether the entire approach is wrong.

4. **Detect and break thrashing loops.** If you've made 2+ fix attempts for the same bug and are
   back where you started, STOP. You are thrashing. The problem is not the code — it's that you lack
   empirical data. Do NOT attempt another code fix. Instead:
   - **Instrument first:** Add temporary `console.log` statements at every critical junction in the
     failing code path. Log variable values, not just "got here" messages.
   - **Inspect the DOM:** If the UI is frozen/broken, check DevTools Elements panel and Computed
     styles. Is `pointer-events: none` set? Is there an overlay? Is the element even in the DOM?
   - **Check the Performance tab:** Is the main thread blocked? Is there an infinite loop?
   - **Verify your assumptions:** You've been theorizing without proof. One empirical observation
     beats ten code-reading sessions. Get hard data before writing any more fix code.

   Only after you have concrete evidence of what's actually happening should you attempt another fix.

5. **Never commit a fix without user verification.** When the user reports a visual or behavioral
   bug, do NOT commit the fix immediately after writing code. The user must confirm the fix actually
   works first. Write the fix, then ask the user to verify. Only commit after they confirm. This
   applies to all UI/visual fixes and any fix where the root cause was uncertain. A wrong fix baked
   into a commit is worse than no commit — it pollutes history and creates false confidence.

## Documentation

See `/info/` for detailed design docs including:

- `architecture.md` - System design overview
- `deep-wallwars-integration.md` - AI engine integration
- `bot_protocol_3.md` - Custom bot communication protocol
- `universal_model.md` - Latest training model documentation

## Deployment

- **Platform:** Fly.io (https://wallgame.fly.dev)
- **Database:** Neon PostgreSQL
- **Config:** `fly.toml`, `Dockerfile`
- **Migrations:** Auto-run via `release_command` in fly.toml

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nmamano) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
