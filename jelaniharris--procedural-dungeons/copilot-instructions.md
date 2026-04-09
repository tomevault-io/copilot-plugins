## procedural-dungeons

> Browser-based 3D dungeon crawler. Procedurally generated floors, turn-based movement, tile-based grid. Two modes: **daily** (seeded by date) and **adventure** (random seed).

# CLAUDE.md — Tower of Greed (Procedural Dungeons)

## Project Overview
Browser-based 3D dungeon crawler. Procedurally generated floors, turn-based movement, tile-based grid. Two modes: **daily** (seeded by date) and **adventure** (random seed).

Live: https://tower-of-greed.vercel.app

## Commands
```bash
yarn dev          # Start dev server (localhost:3000)
yarn build        # Production build
yarn lint         # ESLint
yarn tsc --noEmit # Type-check without building
ANALYZE=true yarn build  # Bundle analysis
```

No test framework is configured.

## Tech Stack
- **Next.js 13** (App Router) + **React 18** + **TypeScript 5**
- **Three.js** + **React Three Fiber** + **@react-three/drei** — 3D rendering
- **Zustand** — state management (modular slices pattern)
- **tRPC** + **@tanstack/react-query** — type-safe API
- **Tailwind CSS** + **Sass** — styling
- **MongoDB** (primary DB) + **AWS DynamoDB** (alt)
- **anime.js** + **react-spring** — animations

## Path Aliases
`@/*` → `./src/*` (configured in tsconfig.json)

## Key Files
| File | Role |
|------|------|
| `src/components/scenes/DungeonScene.tsx` | Main game loop (~900 LOC): player movement, AI, item handling |
| `src/stores/mapSlice.ts` | Map generation (BSP), tiles, items, containers, hazards (~1350 LOC) |
| `src/stores/playerSlice.ts` | Player state: health, energy, inventory, status effects (~784 LOC) |
| `src/stores/enemySlice.ts` | Enemy AI, movement, spawning (~492 LOC) |
| `src/stores/useStore.ts` | Zustand store combiner; seed assignment + game init |
| `src/stores/stageSlice.ts` | Floor progression, save/load, provisions (~330 LOC) |
| `src/components/Game.tsx` | Root game context, keyboard controls, object registry (~190 LOC) |
| `src/components/types/GameTypes.tsx` | Enums: tiles, items, enemies, directions, hazards (~491 LOC) |
| `src/components/types/EventTypes.tsx` | Pub/Sub event key constants |
| `src/utils/gridUtils.ts` | Pathfinding, area calculations |
| `src/utils/randomGenerator.ts` | Seeded RNG for procedural generation |
| `src/utils/pubSub.ts` | Event publish/subscribe system |

## Architecture

### State (Zustand Slices)
`useStore.ts` combines: `MapSlice`, `PlayerSlice`, `StageSlice`, `HazardSlice`, `GeneratorSlice`, `AudioSlice`, `EnemySlice`.

Each slice file exports:
1. An interface (`MapSlice`, `PlayerSlice`, etc.)
2. A `createXSlice` function

### Scenes
`page.tsx` → `SceneManager` → one of:
- `MenuScene` — title screen
- `EmbarkScene` — pre-run provisions selection
- `DungeonScene` — active gameplay

### Event System
Custom pub/sub (`utils/pubSub.ts`). Event types defined in `EventTypes.tsx` as SCREAMING_SNAKE_CASE constants (e.g., `PLAYER_MOVED`, `OVERLAY_TEXT`).

### Rendering
Three.js entities: `GameObject` (base), `MoveableObject` (animated), `Player`. All managed via Zustand + React Three Fiber.

## Naming Conventions
- Components/types: `PascalCase`
- Hooks: `useCamelCase`
- Utilities: `camelCase`
- Enum values: `SCREAMING_SNAKE_CASE` (e.g., `TILE_WATER`, `ITEM_COIN`, `DIR_NORTH`)
- Slice creators: `createXSlice`

## Code Style (Prettier)
- 2-space indentation, single quotes, semicolons
- `prettier-plugin-organize-imports` — auto-sorts imports
- `prettier-plugin-tailwindcss` — sorts Tailwind classes

## Known Issues (from audits/complexity-audit.md)
- `async/forEach` not awaited in `enemySlice.ts` — AI may not move correctly
- Direct state mutation in `enemySlice.ts` `aiMove`
- Stale closures in `DungeonScene` event subscribers
- `DungeonScene.tsx` has 40+ store selectors — consider extracting to custom hooks
- No tests anywhere in the project
- `localStorage` calls unprotected (can crash in SSR/private browsing)

## Environment Variables
Copy `.env.example` → `.env.local`:
```
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
NEXT_PUBLIC_API_URL=
DYNAMO_TABLE_NAME="greedtower-dev"
MONGODB_URI=
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jelaniharris)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jelaniharris)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
