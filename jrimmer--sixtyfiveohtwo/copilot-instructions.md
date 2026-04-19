## sixtyfiveohtwo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Root-level commands
npm install              # Install deps and build all games (postinstall)
npm start                # Start production server on port 3000
npm run dev              # Start server with hot reload (--watch)
npm run build            # Build all games (Telengard + Sabotage)
npm run build:telengard  # Build Telengard only
npm run build:sabotage   # Build Sabotage only

# Telengard (React + Vite)
cd telengard && npm run dev      # Dev server at http://localhost:5173
cd telengard && npm run build    # Production build
cd telengard && npm run lint     # ESLint

# Sabotage (React + Vite)
cd sabotage/sabotage-web && npm run dev    # Dev server at http://localhost:5174
cd sabotage/sabotage-web && npm run build  # Production build
cd sabotage/sabotage-web && npm run lint   # ESLint

# Proving Grounds (Express + SQLite)
cd provinggrounds && npm run db:init  # Initialize database (required before first run)
cd provinggrounds && npm run db:seed  # Seed game data
cd provinggrounds && npm test         # Run tests (node --test)

# TPro BBS (Express + SQLite)
cd tprobbs && node src/db/init.js     # Reset database (deletes all data)
cd tprobbs && node src/db/ensure.js   # Safe init (creates/seeds only if needed)
npx playwright test tprobbs/tests/e2e.spec.js  # E2E tests
```

## Architecture

This is a monorepo containing four classic Apple II games ported to modern web:

```
server.js                 # Express server routing to all games
├── /telengard/          # React SPA (dungeon crawler RPG)
├── /sabotage/           # React SPA (arcade shooter)
├── /provinggrounds/     # Express SSR (BBS door game)
└── /tprobbs/            # Express SSR (BBS system recreation)
```

### Server Routing (`server.js`)

- `GET /` - Landing page with game menu (EJS template)
- `GET /telengard/*` - Static SPA serving from `telengard/dist/`
- `GET /sabotage/*` - Static SPA serving from `sabotage/sabotage-web/dist/`
- `/provinggrounds/*` - Express routes + Socket.IO
- `/tprobbs/*` - Express routes for TPro BBS
- `GET /health` - Health check endpoint

### Telengard (React Game)

State management: Zustand store in `src/state/gameState.ts`
Core logic: `src/engine/` - dungeon generation, combat, spells, monsters
Renderers: ASCII (terminal) and Modern UI in `src/renderers/`

Key algorithm: Procedural dungeon uses hash formula `Q = X*XO + Y*YO + Z*ZO + (X+XO)*(Y+YO)*(Z+ZO)` for 200x200x50 grid

### Sabotage (React Game)

Game loop: `src/engine/update.ts` - ported from 6502 assembly
Rendering: Canvas API with 3 visual modes (Retro/Clean/Modern)
Constants: `src/constants/game.ts` - values from original disassembly

Disassembly reference: `sabotage/Sabotage.dis65` (6502bench SourceGen project)

### Proving Grounds (Express App)

Routes: `src/routes/` - auth, combat, stores, games, boards
Database: SQLite via better-sqlite3, schema in `src/db/schema.sql`
Templates: EJS in `views/`
Real-time: Socket.IO for WebSocket features

### TPro BBS (Express App)

Routes: `src/routes/` - auth, main, boards, email, combat, games, stores, gangs
Database: Separate SQLite database (`tprobbs/data/tprobbs.db`)
Templates: EJS in `views/pages/`
Styling: Shared `terminal.css` from Proving Grounds
Features: Screen themes (green/amber/white phosphor), baud rate simulation, keyboard shortcuts

## Tech Stack

| Game | Frontend | State | Build |
|------|----------|-------|-------|
| Telengard | React 19 + TS | Zustand | Vite 7 |
| Sabotage | React 19 + TS + Canvas | Local state | Vite 7 |
| Proving Grounds | EJS templates | Express session | N/A |
| TPro BBS | EJS templates | Express session | N/A |

Shared: Express 5, better-sqlite3, Socket.IO, Helmet

## Testing

Run tests per-game only, not the full suite:

```bash
cd telengard && npm run lint && npm run build      # Lint + build check
cd sabotage/sabotage-web && npm run lint && npm run build
cd provinggrounds && npm test                       # Node.js test runner
npx playwright test tprobbs/tests/e2e.spec.js      # TPro BBS E2E tests
```

## Game-Specific Guidance

Each game has its own `CLAUDE.md` with detailed architecture:
- `telengard/CLAUDE.md` - Dungeon algorithms, spell system, combat formulas
- `sabotage/CLAUDE.md` - Memory layout, 6502 entry points, copy protection details
- `tprobbs/CLAUDE.md` - Combat formulas, XP system, database schema, testing guide

## Environment Variables

Copy `.env.example` to `.env`:

```bash
PORT=3000
NODE_ENV=development
SESSION_SECRET=your-secret-key-here  # Required in production
DATABASE_PATH=./provinggrounds/data/provinggrounds.db
TPROBBS_DATABASE_PATH=./tprobbs/data/tprobbs.db
```

## Deployment

Railway deployment configured via `railway.json` with health check at `/health`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jrimmer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
