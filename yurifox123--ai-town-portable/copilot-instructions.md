## ai-town-portable

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

AI生态小镇 (AI Eco Town) is a multi-agent simulation system based on Stanford's "Generative Agents" research paper. It simulates autonomous AI agents with memory, reflection, and planning capabilities living in a virtual 2D world.

**Architecture Note:** The project was refactored to a web-first architecture. The simulation now runs entirely in the browser, and the Node.js server is a lightweight `http.createServer` app for LLM API proxying and static file serving. `ARCHITECTURE.md` is outdated (still describes the old CLI architecture) — rely on this file and the source code instead.

## Common Commands

```bash
# Initialize runtime folders and .env
npm run setup

# Check whether this computer can run the project
npm run doctor

# Start web server (default) - runs on port 3061
npm start

# Development with hot reload (uses tsx watch)
npm run dev

# Build TypeScript
npm run build

# Run tests (vitest is installed but no test files exist yet)
npm test

# Lint
npm run lint

# Stop server (uses the local /api/stop endpoint; defaults to port 3061)
npm run stop
```

Note: vitest uses default configuration (no `vitest.config.*` file exists). The `data/` directory contains the SQLite database file (`ai-town.db`) and save files.
Prefer the npm scripts above as the source of truth; the Windows `.bat` helpers are legacy wrappers and can drift from the current scripts.
Use `npm ci` on fresh machines because `package-lock.json` is committed and is the reproducible dependency source of truth.

## Testing

There are no formal unit/integration tests. vitest is installed but no test files exist yet. When modifying frontend code, verify visually by running the server and opening `http://localhost:3061`.

## Architecture

### Current Architecture (Web-First)

The system has been refactored from a CLI-based simulator to a browser-based simulation:

1. **Server** (`src/server/`) - raw `http.createServer` HTTP server with manual route matching:
   - `index.ts` - Entry point, mounts middleware and routes
   - `routes/llm.ts` - Proxies LLM requests to configured provider
   - `routes/agents.ts` - Agent CRUD and actions
   - `routes/memories.ts` - Memory retrieval and storage
   - `routes/reflections.ts` - Reflection generation
   - `routes/map.ts` - Map data and building management
   - `routes/state.ts` - Simulation state persistence (including snapshot save/load)
   - `routes/sprites.ts` - Character sprite/portrait upload and listing
   - `middleware/json.ts` - JSON body parser
   - `middleware/multipart.ts` - Multipart form handling

2. **Database** (`src/server/db/`) - SQLite via better-sqlite3:
   - `connection.ts` - Database connection singleton
   - `schema.ts` - Table definitions (agents, memories, embeddings, reflections, areas, area_cells, simulation_state, dialogues, reflection_sources)

3. **Browser-Based Simulation** (`public/js/`) - Frontend organized by domain:
   - `core/agent.js` - `Agent` class: perception, decision-making, action execution
   - `core/simulator.js` - `WorldSimulator` class: 2D grid simulation with tick-based timing, pollution, resources, building levels
   - `core/memory.js` - `MemorySystem` class: three-layer memory (observations, reflections, plans)
   - `core/pathfinder.js` - A\* pathfinding with terrain collision
   - `core/personality.js` - Prompt construction and personality weight calculation
   - `core/prompts.js` - All LLM prompt templates in one place (imported by personality.js and memory.js)
   - `core/game-config.js` - Centralized game tuning constants (thresholds, rates, multipliers, pollution, building levels)
   - `app/app.js` - Main simulation logic, UI, event handling (canvas renderer)
   - `app/llm-client.js` - Communicates with backend `/api/llm/chat` endpoint
   - `assets/asset-config.js` - Sprite paths and display sizes
   - `assets/image-loader.js` - Asset loading manager
   - `editor/building-editor.js` - Map editing tools

   **Module dependency graph** (ES modules, all imported by `app/app.js`):

   ```
   app/app.js → core/simulator.js → core/agent.js → core/memory.js → core/prompts.js
                                            → core/pathfinder.js
                                            → core/personality.js → core/prompts.js
                                            → core/game-config.js
                 → app/llm-client.js
                 → assets/image-loader.js
                 → assets/asset-config.js
   ```

### Movement System

Agent movement is now independent of the simulation tick:

- **Tick interval** (`TICK_INTERVAL_MS`): Controls decision-making frequency (default: 5000ms)
- **Move interval**: 200ms per grid cell (independent timer in Agent)
- Agents make new decisions every 50 ticks OR when reaching their destination
- Movement uses A\* pathfinding (`pathfinder.js`) with terrain collision detection
- Path recalculates dynamically when blocked

### Memory Retrieval Algorithm

The system uses a three-dimensional weighted scoring for memory retrieval:

```
score = relevance × 0.6 + recency × 0.2 + importance × 0.2
```

Where:

- Relevance = cosine similarity between query and memory embeddings
- Recency = exponential decay based on hours since creation
- Importance = 1-10 score normalized to 0-1

### Agent Lifecycle

```
initialize() → perceive() → decide() → executeAction() → (loop)
```

Reflection triggers when memory count exceeds 100, generating high-level insights that are added back to the memory stream.

### Survival Attributes System

Each Agent has three survival attributes that affect behavior:

- **Health** (0-100): Affected by hunger, sleep, pollution, and activities
  - Sleeping at home restores health (+10/hour)
  - Hunger below 20 causes health loss
  - Sleep deprivation penalty: 1 day (-10), 2 days (-50), 3 days (health → 0)
  - High pollution (≥70) causes gradual health damage; critical pollution (≥90) adds extra damage

- **Fullness** (0-100): Hunger level, decreases over time
  - Base consumption: ~0.6/hour
  - Moving adds: +0.3/hour
  - Working adds: +0.3/hour
  - Eating at buildings restores fullness

- **Green Points** (-10000 to 10000000): Currency system
  - Earned by working at cafes/convenience stores (scaled by building level)
  - Spent on food and services
  - Starting amount: 10 points

### Pollution & World Resources System

The town has a global pollution meter and five world resources that drive the economy:

- **Pollution** (0-100): Starts at 50. Increases by 1 per game day naturally. If it reaches 100, the game ends.
  - Agents can reduce pollution by interacting with the "许愿池" (wishing well / cleanup site).
  - Factories switch from polluting to cleaning once the town reaches building level 4 (resource value ≥50).

- **World Resources**:
  - `techTheory` — increased by working at 实验室 (laboratory)
  - `techProduction` — increased by working at 工厂 (factory), 仓库 (warehouse), 田地 (field), 物资基地 (supply base)
  - `materialValue` — increased by interacting with 田地 (farming) and 物资基地
  - `knowledgeReserve` — increased by working at 图书馆 (library)
  - `foodStock` — increased by 田地 production, capped at 200

Resource accumulation is calculated per tick and multiplied by the agent's work hours.

### Building Level System

Buildings have 5 levels based on the town's accumulated resource value:

| Level | Threshold | incomeMult | pollutionMult | effectMult | costMult |
| ----- | --------- | ---------- | ------------- | ---------- | -------- |
| 1     | —         | 1.0        | 0.8           | 1.0        | 1.0      |
| 2     | ≥10       | 1.3        | 1.0           | 1.2        | 1.2      |
| 3     | ≥30       | 1.6        | 1.3           | 1.5        | 1.4      |
| 4     | ≥50       | 2.0        | 1.5           | 1.8        | 1.6      |
| 5     | ≥100      | 2.5        | 0.5           | 2.0        | 2.0      |

Level affects income earned, pollution generated, service effects, and purchase costs. Factories uniquely switch to _reducing_ pollution at level 4+.

### Terrain System

The world includes impassable terrain that agents must navigate around:

- **wall**: Map boundary, impassable
- **river**: Winding waterway, impassable
- **fence**: Decorative barriers (e.g., around park), impassable
- **gate**: Passable entry points (map edges, park entrance)
- **bridge**: Passable river crossings

Collision detection in `agent.js:moveOneStep()` prevents agents from entering impassable cells and triggers path recalculation.

### Building Services

Buildings provide services that agents can use:

```typescript
{ name: string, cost: number, fullness?: number, health?: number, income?: number, description: string }
```

**Service Types:**

- **Food services**: Restore fullness (coffee +5, snacks +10, meals +25-50)
- **Sleep services**: Restore health (+10), only available at home
- **Work services**: Earn green points (15-25/hour), available at cafe/shop
- **Recreation**: Park activities (walking, exercise)
- **Cleanup**: Reduce town pollution (许愿池)
- **Resource production**: Increase world resources (实验室, 工厂, 田地, 图书馆, 物资基地)

**Action Types:**

- `MOVE` - Navigate to target position
- `TALK` - Conversations with nearby agents
- `WAIT` - Idle action
- `SLEEP` - Return home and sleep (nighttime priority)
- `WORK` - Work at building to earn points or resources
- `BUY` - Purchase food/service at nearby building
- `CLEANUP` - Reduce pollution at cleanup site

### Decision Priority System

Agent decisions follow a strict priority (highest to lowest):

1. Sleep deprivation (2+ days) → must SLEEP
2. Health < 30 → prioritize rest
3. Nighttime (22:00-6:00) → should SLEEP
4. Starving + has money → BUY food
5. Starving + no money → WORK first
6. Low points → consider WORK
7. Hungry + has money → seek food

Town context (pollution level, resource shortages) is injected into the LLM prompt so agents can react to global crises.

## LLM Configuration

The server proxies all LLM requests through `/api/llm/chat`. The current implementation uses the `custom` provider path end-to-end and persists `LLM_PROVIDER=custom` at runtime.

Key variables:

| Variable | Purpose |
| -------- | ------- |
| `CUSTOM_API_KEY` | API key used by the backend proxy |
| `CUSTOM_MODEL` | Chat model name |
| `CUSTOM_ENDPOINT` | Anthropic/OpenAI-compatible chat endpoint |
| `CUSTOM_RESPONSE_PATH` | JSON path used to extract text from the upstream response |
| `CUSTOM_API_KEY_HEADER` | Optional auth header override (`api-key`, `x-api-key`, or `authorization`) |
| `CUSTOM_ANTHROPIC_VERSION` | Optional override for the `anthropic-version` header |

Default: `custom` provider with **Kimi K2.5** via Alibaba Cloud DashScope. The custom provider uses Anthropic-style headers and extracts responses via `CUSTOM_RESPONSE_PATH` (e.g., `content[1].text` for Kimi's thinking+text format).

Optional: `CUSTOM_EMBEDDING_ENDPOINT` / `CUSTOM_EMBEDDING_RESPONSE_PATH` for a separate embedding service.

## Project Structure

```
src/
├── server/
│   ├── index.ts            # Server entry point, mounts routes
│   ├── db/
│   │   ├── connection.ts   # SQLite connection singleton
│   │   └── schema.ts       # Table definitions + migrations
│   ├── routes/
│   │   ├── llm.ts          # LLM proxy endpoints
│   │   ├── agents.ts       # Agent CRUD
│   │   ├── memories.ts     # Memory operations
│   │   ├── reflections.ts  # Reflection generation
│   │   ├── map.ts          # Map/building management
│   │   ├── state.ts        # Simulation state + snapshot save/load
│   │   └── sprites.ts      # Character sprite upload and listing
│   └── middleware/
│       ├── json.ts         # JSON body parser
│       └── multipart.ts    # Multipart form handling

public/
├── index.html
├── styles.css
├── js/
│   ├── core/               # Simulation logic
│   │   ├── agent.js
│   │   ├── simulator.js
│   │   ├── memory.js
│   │   ├── pathfinder.js
│   │   ├── personality.js
│   │   ├── prompts.js      # All LLM prompt templates
│   │   └── game-config.js  # Centralized tuning constants
│   ├── app/                # UI and main entry
│   │   ├── app.js          # Main entry, canvas renderer (~76KB)
│   │   └── llm-client.js
│   ├── assets/             # Asset management
│   │   ├── asset-config.js
│   │   ├── image-loader.js
│   │   └── sprite-crop-tool.js
│   ├── editor/             # Map editor
│   │   └── building-editor.js
│   └── tools/              # Dev tools
│       └── anim-test.js
└── assets/                 # Images and sprites

data/                       # Runtime data (gitignored)
├── ai-town.db              # SQLite database
└── saves/                  # Save files

ARCHITECTURE.md
```

## Environment Variables

Critical variables (from `.env.example`):

- `LLM_PROVIDER` - currently forced to `custom` by the runtime config flow
- `CUSTOM_API_KEY` / `CUSTOM_ENDPOINT` / `CUSTOM_RESPONSE_PATH` - For custom provider
- `CUSTOM_EMBEDDING_ENDPOINT` / `CUSTOM_EMBEDDING_RESPONSE_PATH` - Optional embedding service
- `DATABASE_URL` - SQLite database path (default: `./data/ai-town.db`)
- `CHROMA_DB_PATH` - ChromaDB path (legacy, currently unused)
- `PORT` - Server port (default: 3061, auto-increments if in use)
- `TICK_INTERVAL_MS` - Simulation tick interval (default: 5000ms)
- `WORLD_WIDTH` / `WORLD_HEIGHT` - Map dimensions (default: 50x50)
- `MAX_AGENTS` - Maximum number of agents (default: 10)
- `TIME_SCALE` - Game time speed (default: 60, meaning 1 real second = 1 game minute)

## Map Editor

The web interface includes a full map editor accessible via "编辑地图" button:

**Tools:**

- **Select**: Click to select/move buildings
- **Ground/Path**: Paint terrain tiles (grass, path, water)
- **Building**: Place new buildings
- **Eraser**: Remove buildings

**Features:**

- Drag buildings to reposition
- Edit building properties (name, size, obstacle flag)
- Import/export map data as JSON
- Undo/redo with Ctrl+Z/Ctrl+Y
- Delete selected building with Delete key

## Web Frontend

Run `npm start` and open `http://localhost:3061`. The UI supports real-time simulation visualization, agent details (click to view memories), event log, and a map editor.

### API Endpoints

The server exposes these endpoints:

- `GET/PUT /api/llm/config` - Read/update persisted LLM config
- `POST /api/llm/config/test` - Test the current LLM config
- `POST /api/llm/chat` - Proxy LLM requests (messages, options)
- `POST /api/llm/embedding` - Get text embeddings
- `GET/POST /api/agents` - Agent CRUD list/create
- `GET/PATCH/DELETE /api/agents/:id` - Read/update/delete a single agent
- `GET/POST /api/agents/:id/memories` - List/create memories
- `DELETE /api/agents/:id/memories/:memoryId` - Delete one memory
- `GET/POST /api/agents/:id/reflections` - List/create reflections
- `GET/POST/PUT /api/map/areas` - Area CRUD / full replace
- `DELETE /api/map/areas/:id` - Delete one area
- `GET/PUT /api/state` - Read/update simulation state
- `POST /api/state/snapshot` - Full world save (agents, memories, reflections, areas, state)
- `GET /api/state/snapshot` - Full world load
- `GET /api/state/snapshots` - List saved snapshots
- `GET /api/sprites/list` - List available sprites and portraits
- `POST /api/sprites` - Upload sprite/portrait image
- `POST /api/sprites/batch` - Upload multiple sprite frames
- `POST /api/sprites/config` - Update asset-config for a character
- `POST /api/stop` - Shut down the server

## TypeScript Configuration

Uses `moduleResolution: "bundler"` with path mapping `@/*` → `src/*`. The `dist/` directory contains compiled output.

---
> Source: [yurifox123/ai-town-portable](https://github.com/yurifox123/ai-town-portable) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
