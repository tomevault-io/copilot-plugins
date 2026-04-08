## action-idle

> All non-presentation code must be easily portable to Godot. Everything in `/src/core/` uses interfaces and pure TypeScript with zero React/browser dependencies.

# Total Idle - Project Context

## Primary Constraint

All non-presentation code must be easily portable to Godot. Everything in `/src/core/` uses interfaces and pure TypeScript with zero React/browser dependencies.

**ESLint enforces this**: The `eslint.config.js` has a rule blocking React imports in `/src/core/`. Violations will fail linting.

## Development Status

The game has two major systems being developed in phases:

| System | Status | Description |
|--------|--------|-------------|
| **Battle System** | 🟢 Active | Auto-battler with formations, targeting AI, combat |
| **Economy System** | 🟡 Dormant | Idle progression, upgrades, currency (future integration) |

Current focus: **Battle System** - unit spawning, formations, combat mechanics.

## Architecture

### Folder Structure

```
/src
├── core/                    # Pure TypeScript - portable to Godot
│   ├── battle/             # 🟢 BattleEngine, combat AI, unit types
│   ├── engine/             # 🟡 GameEngine, Formulas (idle economy)
│   ├── persistence/        # 🟡 IPersistenceAdapter, SaveManager
│   ├── physics/            # 🟡 Vector2, Zoom (math utilities)
│   ├── theme/              # 🟢 Centralized colors (faction, UI, arena)
│   ├── types/              # GameState, Upgrade interfaces
│   └── utils/              # BigNumber (break_infinity.js wrapper)
├── hooks/                   # React integration layer
│   ├── useBattle.ts        # 🟢 Battle engine bridge
│   ├── useGameLoop.ts      # requestAnimationFrame tick loop
│   └── useGameState.ts     # 🟡 Economy engine bridge
├── components/              # React + Tailwind UI
│   ├── battle/             # 🟢 BattleView, BattleCanvas
│   ├── ui/                 # 🟡 StatsDisplay, UpgradeShop
│   └── App.tsx
└── data/
    ├── upgrades.json       # 🟡 Upgrade definitions (economy - dormant)
    ├── units/              # 🟢 Unit definitions (warrior, archer, knight)
    ├── abilities/          # 🟢 Ability definitions (triggers, effects)
    ├── battle-upgrades/    # 🟢 Battle upgrade definitions (stat mods)
    └── battle/             # 🟢 Battle data initializer
/tests                       # Centralized test folder (mirrors src structure)
├── setup.ts                # Vitest setup (jest-dom matchers)
└── core/                   # Tests for /src/core/ modules
    ├── battle/             # BattleEngine, units, modifiers tests
    ├── engine/             # GameEngine, Formulas tests
    └── utils/              # BigNumber tests
```

🟢 = Active development | 🟡 = Dormant (future use)

## Module Design Principles

All code in `/src/core/` must be clean, testable, and directly portable to Godot.

### 1. Single Responsibility

Each module handles ONE concern. Never create god-classes that mix multiple responsibilities:

```typescript
// ✅ Good - focused modules
SelectionManager.ts   // Only selection state
DragController.ts     // Only drag logic
FormationManager.ts   // Only spawn positioning

// ❌ Bad - god-class
BattleCanvas.ts       // Selection + dragging + rendering + input + spawning
```

### 2. Pure Functions Over Stateful Classes

Prefer pure functions that take state and return new state. This maps directly to Godot's functional patterns:

```typescript
// ✅ Good - pure function, easy to port
function selectUnit(state: SelectionState, unitId: string): SelectionState {
  return { selectedIds: [unitId] };
}

// ❌ Avoid - hidden state makes porting harder
class SelectionManager {
  private selectedIds: string[] = [];
  selectUnit(unitId: string) { this.selectedIds = [unitId]; }
}
```

### 3. Thin Presentation Layer

React components must be thin wrappers. All logic lives in `/src/core/`:

```typescript
// ✅ Component just bridges input to core
const handleMouseDown = (e: MouseEvent) => {
  const pos = getMousePos(e);                    // Platform-specific (React)
  const unit = findUnitAtPosition(pos, units);   // Core module (portable)
  onSelectUnit(selectUnit(state, unit?.id));     // Core module (portable)
};

// ❌ Logic embedded in component - not portable
const handleMouseDown = (e: MouseEvent) => {
  for (const unit of units) {
    if (Math.hypot(pos.x - unit.x, pos.y - unit.y) < unit.size) {
      setSelectedIds([unit.id]);  // React-specific, can't port
    }
  }
};
```

### 4. Platform-Agnostic Interfaces

Core modules accept simple types (Vector2, arrays), not platform-specific events:

```typescript
// Core module - works anywhere
function findUnitAtPosition(pos: Vector2, units: Unit[]): Unit | null

// React converts MouseEvent → Vector2
// Godot converts InputEventMouse → Vector2
```

### 5. Design for Godot Translation

Write core code imagining how it translates to GDScript:

| TypeScript | GDScript Equivalent |
|------------|---------------------|
| `function foo(state, input): NewState` | `static func foo(state, input) -> NewState` |
| `interface State { ... }` | `class_name State extends Resource` |
| `class Registry { static get() }` | `Autoload singleton` |
| `tick(delta: number)` | `_process(delta: float)` |
| `new Vector2(x, y)` | `Vector2(x, y)` |
| `callback: (event) => void` | `signal event_occurred` |

### 6. Data-Driven Content

Game content lives in JSON, loaded by registries:
- `/src/data/units/*.json` - Unit definitions
- `/src/data/abilities/*.json` - Ability definitions
- `/src/data/battle-upgrades/*.json` - Upgrade definitions

This maps to Godot's Resource system - JSON becomes `.tres` files.

### 7. Interface-First Design

For platform-specific features (persistence, physics, rendering), define interfaces in core and implement outside:

```
/src/core/
├── persistence/
│   ├── IPersistenceAdapter.ts    # Interface - what Godot must implement
│   └── SaveManager.ts            # Uses interface, not concrete class
```

```typescript
// ✅ Good - interface defines contract
interface IPersistenceAdapter {
  save(key: string, data: string): Promise<void>;
  load(key: string): Promise<string | null>;
}

// ✅ Good - core depends on interface
class SaveManager {
  constructor(private adapter: IPersistenceAdapter) {}
}

// ❌ Bad - core depends on concrete implementation
class SaveManager {
  private adapter = new LocalStorageAdapter();  // Hardcoded!
}
```

**Index files export interfaces, not implementations:**

```typescript
// ✅ Good - /src/core/persistence/index.ts
export type { IPersistenceAdapter } from './IPersistenceAdapter';
export { SaveManager } from './SaveManager';

// ❌ Bad - exports platform-specific implementation
export { LocalStorageAdapter } from './LocalStorageAdapter';  // Don't!
```

Platform implementations live OUTSIDE core:
- `/src/adapters/LocalStorageAdapter.ts` - React/browser implementation
- Godot implements same interface with `FileAccess`

### 8. Dependency Injection

Never instantiate platform-specific classes inside core. Pass dependencies in:

```typescript
// ✅ Good - dependency injected (in React hook or Godot scene)
const adapter = new LocalStorageAdapter();  // Platform layer creates this
const saveManager = new SaveManager(adapter);  // Core receives interface

// ❌ Bad - hardcoded inside core module
class GameManager {
  private saveManager = new SaveManager(new LocalStorageAdapter());
}
```

This lets Godot swap implementations without touching core:

```gdscript
# Godot implementation
var adapter = GodotFileAdapter.new()
var save_manager = SaveManager.new(adapter)
```

### 9. Entity Architecture (Scene/Node Pattern)

**Current approach: Godot-style Scene/Node Pattern**

The battle system uses a Godot-compatible entity architecture:

- **Entities** (`UnitEntity`, `ProjectileEntity`) handle their own behavior
- **BattleWorld** manages entity lifecycle and queries (like Godot's SceneTree)
- **BattleEngine** orchestrates the battle, delegates to BattleWorld
- **Events** map directly to Godot signals

```typescript
// Entity handles its own behavior
class UnitEntity extends BaseEntity implements IEntity, IEventEmitter {
  update(delta: number): void {
    this.updateTargeting();
    this.updateCombat(delta);
    this.updateMovement(delta);
  }
}

// World manages entities (like Godot main scene)
class BattleWorld {
  update(delta: number): void {
    for (const entity of this.entities) {
      entity.update(delta);  // Each entity updates itself
    }
  }
}
```

**Godot mapping:**

| TypeScript | GDScript Equivalent |
|------------|---------------------|
| `IEntity.init()` | `_ready()` |
| `IEntity.update(delta)` | `_process(delta)` |
| `IEntity.destroy()` | `queue_free()` |
| `IEventEmitter.emit()` | `emit_signal()` |
| `IEventEmitter.on()` | `connect()` |
| `BattleWorld` | Main scene or autoload |

**Key classes:**

```
/src/core/battle/
├── IEntity.ts                    # Lifecycle interface + typed events
├── entities/
│   ├── EventEmitter.ts           # Signal system implementation
│   ├── BaseEntity.ts             # Abstract base with common functionality
│   ├── UnitEntity.ts             # Unit with targeting, combat, movement
│   ├── ProjectileEntity.ts       # Projectile with movement, hit detection
│   ├── BattleWorld.ts            # Entity manager (SceneTree equivalent)
│   ├── IBattleWorld.ts           # Query interface for entities
│   └── index.ts
```

**Using the Event System (Godot Signals):**

Entities emit events that map to Godot signals. Subscribe to react to game events:

```typescript
// Subscribe to entity events (like Godot's connect())
const unit = engine.getWorld().getUnitById('unit_1');
unit.on('damaged', (event) => {
  console.log(`${event.entity.id} took ${event.amount} damage`);
});
unit.on('killed', (event) => {
  console.log(`${event.entity.id} was killed by ${event.killer?.id}`);
});
unit.on('attacked', (event) => {
  console.log(`${event.entity.id} attacked ${event.target.id} for ${event.damage}`);
});

// Unsubscribe (like Godot's disconnect())
unit.off('damaged', myHandler);
```

**Events:** See `IEntity.ts` for all typed events (`spawned`, `destroyed`, `damaged`, `killed`, `attacked`, `moved`) and world events (`entity_added`, `entity_removed`). BattleStats uses world events to auto-subscribe to new units.

**Creating New Entity Types:** Extend `BaseEntity`, implement `update(delta)`, use `this.emit()` for events. See `UnitEntity.ts` and `ProjectileEntity.ts` for examples.

**Migration Status: ✅ COMPLETE**

The battle system is fully Godot-ready:

| Layer | Status | Notes |
|-------|--------|-------|
| Core entities | ✅ Done | UnitEntity, ProjectileEntity, BattleWorld |
| Event system | ✅ Done | IEventEmitter with BattleStats consumer |
| Unit definitions | ✅ Done | JSON files in `/src/data/units/` |
| Unit registry | ✅ Done | BattleEngine uses UnitRegistry |
| Stats tracking | ✅ Done | BattleStats subscribes to entity events |
| React rendering | ✅ Done | Uses render data adapter (intentional) |

**For new code:** Use `engine.getWorld()` to access entities directly.
**For Godot port:** Translate entity classes to GDScript scenes. Ignore render data types in `types.ts` - they only exist for React rendering.

### 10. Centralized Configuration

All game constants (spacing, speeds, combat values, etc.) are in `/src/core/battle/BattleConfig.ts`. **Never use magic numbers in code** - always import from BattleConfig.

```typescript
// ❌ Bad - magic numbers in code
if (dist < 1) { ... }
const checkDist = this.size * 3;
if (dot > 0.3) { ... }

// ✅ Good - constants from BattleConfig
import { MIN_MOVE_DISTANCE, DIRECTION_CHECK_MULTIPLIER, PATH_DOT_THRESHOLD } from './BattleConfig';
if (dist < MIN_MOVE_DISTANCE) { ... }
const checkDist = this.size * DIRECTION_CHECK_MULTIPLIER;
if (dot > PATH_DOT_THRESHOLD) { ... }
```

**When adding new gameplay values:**
1. Add constant to `BattleConfig.ts` with descriptive name and JSDoc comment
2. Import and use the constant in your code
3. Never use literal numbers for gameplay-affecting values

### 11. Event System Best Practices

The event system maps to Godot signals. Follow these rules to avoid bugs:

**Single Point of Event Emission:**
```typescript
// ❌ Bad - event emitted in multiple places
class UnitEntity {
  update(delta: number) {
    if (this.health <= 0) this.emit('killed', {...});  // Duplicate!
  }
  takeDamage(amount: number) {
    this.health -= amount;
    if (this.health <= 0) this.emit('killed', {...});  // Original
  }
}

// ✅ Good - event emitted in ONE place only
class UnitEntity {
  update(delta: number) {
    if (this._destroyed) return;  // Skip if already dead
  }
  takeDamage(amount: number) {
    this.health -= amount;
    if (this.health <= 0) {
      this.markDestroyed();
      this.emit('killed', {...});  // Only place this event is emitted
    }
  }
}
```

**Proper Entity Cleanup (Prevent Memory Leaks):**
```typescript
// ❌ Bad - removing entity without cleanup
removeDestroyedEntities() {
  this.entities = this.entities.filter(e => !e.isDestroyed());
  // Memory leak! Listeners not cleared
}

// ✅ Good - call destroy() before removal
removeDestroyedEntities() {
  const destroyed = this.entities.filter(e => e.isDestroyed());
  for (const entity of destroyed) {
    entity.destroy();  // Emits 'destroyed', clears listeners
  }
  this.entities = this.entities.filter(e => !e.isDestroyed());
}
```

**Destroy Method Pattern:**
```typescript
// ✅ Correct destroy() implementation
destroy(): void {
  if (this._cleanedUp) return;  // Prevent double cleanup
  this._cleanedUp = true;
  this.emit('destroyed', { entity: this });
  this.clearAllListeners();  // Prevent memory leaks
}
```

## Key Files

### Battle System (Active)

**Core Modules (Godot-portable):**

| File | Purpose |
|------|---------|
| `src/core/battle/BattleConfig.ts` | **Centralized constants** - all magic numbers in one place |
| `src/core/battle/BattleEngine.ts` | Battle orchestrator - delegates to BattleWorld |
| `src/core/battle/entities/` | **Entity system (Godot Scene/Node pattern)** |
| `src/core/battle/entities/BattleWorld.ts` | Entity manager - lifecycle, queries, separation |
| `src/core/battle/entities/UnitEntity.ts` | Unit entity - targeting, combat, movement AI |
| `src/core/battle/entities/ProjectileEntity.ts` | Projectile entity - movement, hit detection |
| `src/core/battle/entities/BaseEntity.ts` | Abstract base with lifecycle + events |
| `src/core/battle/IEntity.ts` | Entity lifecycle interface (init/update/destroy) |
| `src/core/battle/SelectionManager.ts` | Pure selection state - select, toggle, selectAllOfType |
| `src/core/battle/DragController.ts` | Multi-unit drag with relative positioning, edge clamping |
| `src/core/battle/FormationManager.ts` | Formation templates and spawn positioning |
| `src/core/battle/InputAdapter.ts` | Platform-agnostic input - hit detection |
| `src/core/battle/BoxSelectController.ts` | Box/marquee selection for multiple units |
| `src/core/battle/BoundsEnforcer.ts` | Pure functions for arena boundary enforcement |
| `src/core/battle/shuffle.ts` | Combat shuffle for melee units |
| `src/core/battle/types.ts` | Render data types for React rendering only |
| `src/core/battle/BattleStats.ts` | Battle statistics via entity events |
| `src/core/battle/units/` | Unit definitions, instances, registry, factory |
| `src/core/battle/modifiers/` | Stat modification system with stacking rules |
| `src/core/battle/abilities/` | Trigger-based abilities (on_kill, on_hit, etc.) |
| `src/core/battle/upgrades/` | Battle upgrades with cost scaling |
| `src/core/theme/colors.ts` | Centralized color palette (AC6-inspired industrial mech theme) |
| `src/core/physics/Vector2.ts` | 2D math utilities for positioning |

**React Layer (Thin wrappers - uses render data interface):**

| File | Purpose | Migration |
|------|---------|-----------|
| `src/hooks/useBattle.ts` | React bridge - manages state | Uses `BattleState` with `Unit[]` |
| `src/components/battle/BattleView.tsx` | Main battle UI | Uses render data `Unit` type |
| `src/components/battle/BattleCanvas.tsx` | Canvas rendering | Uses render data `Unit` type |

*Note: React layer intentionally uses render data interface. Core entities are Godot-ready; React rendering doesn't need migration.*

### Economy System (Dormant)

**Core Modules (Godot-portable):**

| File | Purpose |
|------|---------|
| `src/core/engine/IGameEngine.ts` | Interface - what Godot must implement |
| `src/core/engine/GameEngine.ts` | Idle engine - `tick(delta)`, `purchaseUpgrade(id)` |
| `src/core/engine/Formulas.ts` | Pure math: `calculateCost()`, `calculateProduction()` |
| `src/core/persistence/IPersistenceAdapter.ts` | Interface for save/load (swap for Godot) |
| `src/core/types/GameState.ts` | State interfaces (GameState, UpgradeState) |
| `src/core/types/Upgrade.ts` | Upgrade definition interface |
| `src/core/utils/BigNumber.ts` | break_infinity.js wrapper, `formatNumber()` |

**React Layer (Thin wrappers):**

| File | Purpose |
|------|---------|
| `src/hooks/useGameState.ts` | React bridge - owns engine, handles save/load |
| `src/data/upgrades.json` | Upgrade definitions (baseCost, costMultiplier, baseProduction) |

## Formulas

- **Upgrade Cost**: `Cost = BaseCost * CostMultiplier^Level`
- **Production**: `Production = BaseProduction * Level`
- **Currency Gain**: `Gained = TotalProduction * Delta`

## Commands

**⚠️ IMPORTANT: Always use Docker for development, never run npm commands directly.**

### Development (Docker - PREFERRED)

```bash
# Start dev server with hot reload on port 5177
docker compose -f docker-compose.dev.yml up

# Or run in background
docker compose -f docker-compose.dev.yml up -d
```

### npm scripts (run inside Docker container only)

```bash
npm run dev              # Start Vite dev server
npm run build            # TypeScript check + production build
npm run preview          # Preview production build
```

### Quality

```bash
npm run validate         # Full validation (typecheck + lint + test)
npm run typecheck        # TypeScript type checking only
npm run lint             # ESLint check
npm run lint:fix         # ESLint with auto-fix
npm run format           # Prettier format all files
npm run format:check     # Prettier check (CI mode)
```

### Testing

```bash
npm run test             # Vitest watch mode
npm run test:run         # Single test run
npm run test:coverage    # With coverage report
npm run test:core        # Only core tests (Godot-portable code)
```

### Docker

```bash
# Development (hot reload on port 5177)
docker compose -f docker-compose.dev.yml up

# Production (static build on port 3000)
docker compose up -d
docker compose down
docker compose build     # Rebuild after dependency changes
```

## Testing

Tests use **Vitest** with `jsdom` environment for React components.

- Test files live in `/tests/` folder, mirroring `/src/` structure
- Core tests (`tests/core/**/*.test.ts`) are pure TypeScript with no React dependencies
- Use `npm run test:core` to validate Godot-portable code in isolation

### Writing Tests

```typescript
import { describe, it, expect } from 'vitest';
import { Decimal } from '../../../src/core/utils/BigNumber';

describe('MyFunction', () => {
  it('does something', () => {
    expect(result.eq(expected)).toBe(true);  // Use .eq() for Decimal comparison
  });
});
```

## Coding Standards

1. **No React in `/src/core/`** - Never import React, hooks, or browser APIs (enforced by ESLint)
2. **Explicit Delta** - Always pass `delta` (seconds) to time-based functions
3. **Decimal Everywhere** - Use `Decimal` from break_infinity.js for all game numbers
4. **JSON-Driven Content** - Upgrade stats stay in JSON files, not code
5. **Interface-Based Persistence** - `SaveManager` depends on `IPersistenceAdapter`
6. **Minimum Font Size** - Never use `text-xs` in Tailwind. Minimum is `text-sm` for readability
7. **Use Theme Colors** - Never use hardcoded hex colors. Always reference `UI_COLORS` or other theme constants from `src/core/theme/colors.ts`
8. **Use Proper Text Colors** - Use `UI_COLORS.textPrimary` for main text, `UI_COLORS.textSecondary` for labels, and `UI_COLORS.textMuted` for disabled/hint text. Accent colors like `UI_COLORS.accentPrimary` can be used for emphasis but not body text
9. **No Magic Numbers** - All gameplay constants must be in `BattleConfig.ts`. Never use literal numbers like `0.3`, `100`, `2` for gameplay-affecting values
10. **Single Event Emission** - Each event type (`killed`, `damaged`, etc.) must be emitted from exactly ONE place in the code. Never emit the same event from multiple methods
11. **Always Call destroy()** - When removing entities, always call `destroy()` to emit the `destroyed` event and clear listeners. Never just filter entities out of arrays
12. **Check Destroyed State** - In `update()` methods, check `if (this._destroyed) return;` early to skip processing dead entities

## Linting & Formatting

- **ESLint**: Flat config in `eslint.config.js` with TypeScript and React rules
- **Prettier**: Config in `.prettierrc` (single quotes, semicolons, 100 char width)
- **Pre-commit hook**: Husky runs lint-staged on commit (auto-fixes and formats staged files)

## Big Numbers

Uses `break_infinity.js` for numbers up to 10^9e15. Key functions in `src/core/utils/BigNumber.ts`:

- `formatNumber(value: Decimal)` - Human-readable (1.5M, 2.3B, 1.2e15)
- `serializeDecimal()` / `deserializeDecimal()` - For save/load

## Color System

All colors are in `src/core/theme/colors.ts` - never use hardcoded hex values. Inspired by Armored Core 6 industrial mech aesthetic with dark panels, metallic surfaces, and high-contrast accent colors. Includes utility functions (`hexToRgba`, `getUnitColor`). For Godot, convert hex to `Color.html()` or `Color8()`.

## Godot Migration Path

1. Port `/src/core/` to GDScript/C# (syntactic translation)
2. Convert `colors.ts` to GDScript using `Color.html()` or `Color8()`
3. Implement `IPersistenceAdapter` using Godot's `ConfigFile`
4. Replace React components with Godot Control nodes
5. Use `_process(delta)` instead of `requestAnimationFrame`

Run `npm run test:core` to validate core code independently of React.

### Scaling System Note

The codebase has a manual scaling system in `BattleConfig.ts` using `scaleValue(baseValue, arenaHeight)`. This exists because React/browser doesn't have built-in viewport scaling.

**For Godot, you have two options:**

| Option | Approach | Recommendation |
|--------|----------|----------------|
| **A (Recommended)** | Use Godot's built-in stretch mode | Set project base resolution to 600px height, use `stretch_mode = "canvas_items"`. Remove `scaleValue()` calls and use `BASE_` constants directly. |
| **B** | Keep manual scaling | Set `stretch_mode = "disabled"`. Use `scale_value(BASE_X, get_viewport().size.y)` as-is. |

**Why Option A is better:**
- Godot's stretch system handles scaling automatically
- Less code to maintain
- Avoids risk of double-scaling (Godot + manual)

**If using Option A**, the `scaleValue()` function and `REFERENCE_ARENA_HEIGHT` constant can be removed during the Godot port. The `BASE_` prefixed constants become the actual values used directly.

## Save System

- Autosaves every 30 seconds
- Offline earnings calculated on load (capped at 1 hour)
- Save key: `action_idle_save` in localStorage
- Version field in saves for future migrations

## CI/CD

GitHub Actions workflow (`.github/workflows/ci.yml`) runs on push/PR to main:

1. Type checking
2. Linting
3. Format checking
4. Tests with coverage
5. Build verification

## Visual Debugging

Use Puppeteer to take screenshots and verify UI changes:

```bash
# Take a screenshot of the running app (saves to screenshots/ folder)
node scripts/screenshot.cjs http://localhost:5177 screenshots/my-screenshot.png
```

The script (`scripts/screenshot.cjs`):

- Disables cache to get fresh content
- Forces a page reload before capturing
- Waits for rendering to complete

Use screenshots to:

- Verify layout changes before telling the user
- Debug visual issues
- Confirm styling updates

The dev server runs on port **5177** (configured in `vite.config.ts` with `strictPort: true`).

## Battle System

Units can have both melee and ranged attacks. They automatically switch based on distance:

- **Ranged mode**: Used when target is far (if unit has ranged attack)
- **Melee mode**: Used when in close range (combat shuffle activates)

**Design**: Pure melee units can gain ranged attacks via upgrades/equipment later.

### Combat Flow

1. Units spawn in deployment zones (25% height each)
2. Player can drag allied units before battle starts
3. Battle starts → units acquire targets and move
4. Ranged units fire projectiles, switch to melee when enemies close
5. Melee units shuffle side-to-side while fighting
6. Dead units removed, battle continues until one side eliminated

## Battle Arena Coordinate System

The battle arena uses standard screen coordinates:
- **Y = 0** is at the **top** of the screen (enemy side)
- **Y increases downward** toward the bottom (allied side)

When positioning units:
- **Lower Y = closer to enemy** (front line)
- **Higher Y = further from enemy** (back line)

## Documentation

| Document | Description |
|----------|-------------|
| `docs/GAME_DESIGN.md` | Game design document - mechanics, balance, progression |
| `docs/PHYSICS.md` | Physics engine architecture and Godot migration guide (outdated - IPhysicsEngine removed) |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ebergstedt)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ebergstedt)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
