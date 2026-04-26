## fast-ferocious-fandango-foolish-fugitives

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Fast Ferocious Fandango: Foolish Fugitives** - A mobile-first physics-based car ramming game. RAM traffic cars off the road for points.

**Current State**: `src/main.ts` is the canonical runtime (Matter.js). The modular architecture in `briefing.md` and `src/game/Game.ts` is legacy/experimental until an intentional migration.

## Golden Goal
A cool, fun-to-play arcade chase with great graphics: rams feel satisfying and fair, the ramp keeps players hooked from 1 car to chaos, and the BUSTED loop keeps runs going without frustration.

## Build Commands (Once TypeScript Project Exists)

```bash
npm run dev          # Start Vite dev server
npm run build        # TypeScript compile + Vite build
npm run test         # Run Vitest tests
npm run test:watch   # Run Vitest in watch mode
npm run typecheck    # TypeScript type check
npm run lint         # ESLint
npm run check        # typecheck + lint + test (all gates)
```

## Architecture

### Design Principles

1. **Functional Core, Imperative Shell** - Pure functions for game logic, side effects isolated
2. **Side-Effect Naming** - All impure functions prefixed with `fx_` (e.g., `fx_render`, `fx_playSound`)
3. **Immutable State** - Game state transforms return new state objects
4. **ECS Pattern** - Entity Component System for game objects

### Directory Structure (Target)

```
src/
├── engine/          # Reusable game engine
│   ├── core/        # Types, ECS, game loop
│   ├── physics/     # Vector math, rigid bodies, collision (pure)
│   ├── render/      # Canvas, camera, particles (fx_ side effects)
│   ├── input/       # Keyboard, touch, gamepad
│   └── audio/       # Sound manager
├── game/            # SPY DRIVER specific
│   ├── components/  # ECS components (VehicleData, AIDriverData, etc.)
│   ├── systems/     # ECS systems (physics.system.ts, ai.system.ts)
│   ├── entities/    # Entity factories (player-car.ts, traffic-car.ts)
│   ├── scenes/      # Game states (menu, game, pause, gameover)
│   └── config/      # Physics config, vehicle stats, difficulty
└── utils/           # Math, seeded random, object pooling
```

### Key Patterns

**Pure vs Impure Separation**:
```typescript
// Pure (no prefix) - returns new state
const stepWorld = (world: PhysicsWorld, dt: number): PhysicsWorld => {...}

// Impure (fx_ prefix) - performs I/O
const fx_renderGame = (state: Readonly<GameState>): void => {...}
```

**Branded Types** for type-safe IDs:
```typescript
type EntityId = number & { readonly __brand: 'EntityId' };
```

**Immutable Physics Bodies**:
```typescript
const applyForce = (body: Body, force: Vec2): Body => ({
  ...body,
  acceleration: add(body.acceleration, scale(force, body.invMass)),
});
```

### Physics System

- 2D rigid body physics with rotation
- AABB broad phase + shape-specific narrow phase collision
- Impulse-based collision resolution with friction
- PIT maneuver detection (hit rear corner = massive spin)

### Game Mechanics

- **Boost**: Swipe up for nitro, drains `boost` meter
- **Brake**: Swipe down, creates skid marks
- **RAM**: Push cars off road with physics collisions
- **PIT Maneuver**: Hit rear corner for bonus spin and points

## TypeScript Config Highlights

- Strict mode with `noUncheckedIndexedAccess`
- Path aliases: `@engine/*`, `@game/*`, `@utils/*`
- ESM native with Vite bundler resolution

## Testing

Pure functions are trivially testable:
```typescript
describe('Vector2D', () => {
  it('adds vectors', () => {
    expect(add(vec2(1, 2), vec2(3, 4))).toEqual({ x: 4, y: 6 });
  });
});
```

## Reference Files

- `docs/plans/2026-01-18-game-design.md` - **Complete game design** (start here!)
- `briefing.md` - TypeScript engine architecture specification
- `spy_driver_physics.html` - Working prototype (vanilla JS, single file)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/franzenzenhofer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
