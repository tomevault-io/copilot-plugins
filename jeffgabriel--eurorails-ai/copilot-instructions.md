## jscodingrules

> Applies general coding rules across all file types to maintain code quality, consistency, and prevent common errors.

- Always verify information before presenting it. Do not make assumptions or speculate without clear evidence.
- Make changes file by file and give me a chance to spot mistakes.
- Never use apologies.
- Avoid giving feedback about understanding in comments or documentation.
- Don't suggest whitespace changes.
- Don't summarize changes made.
- Don't invent changes other than what's explicitly requested.
- Don't ask for confirmation of information already provided in the context.
- Don't remove unrelated code or functionalities. Pay attention to preserving existing structures.
- Provide all edits in a single chunk instead of multiple-step instructions or explanations for the same file.
- Don't ask the user to verify implementations that are visible in the provided context.
- Don't suggest updates or changes to files when there are no actual modifications needed.
- Always provide links to the real files, not the context generated file.
- Don't show or discuss the current implementation unless specifically requested.
- Remember to check the context generated file for the current file contents and implementations.
- Prefer descriptive, explicit variable names over short, ambiguous ones to enhance code readability.
- Adhere to the existing coding style in the project for consistency.
- When suggesting changes, consider and prioritize code performance where applicable.
- Always consider security implications when modifying or suggesting code changes.
- Suggest or include appropriate unit tests for new or modified code.
- Implement robust error handling and logging where necessary.
- Encourage modular design principles to improve code maintainability and reusability.
- Ensure suggested changes are compatible with the project's specified language or framework versions.
- Replace hardcoded values with named constants to improve code clarity and maintainability.
- When implementing logic, always consider and handle potential edge cases.
- Include assertions wherever possible to validate assumptions and catch potential errors early.
- **NO FAILED TESTS ALLOWED**: All tests must pass. Fix any failing tests immediately before proceeding with other work. This is a hard and fast rule that cannot be violated.

Here are our unified coding principles for the JavaScript/TypeScript codebase:

1. **Module System**
- Use ES Modules (`import`/`export`) consistently across all code
- Never mix with CommonJS (`require`/`module.exports`)
```typescript
// ✅ Do:
import { Something } from './something';
export class MyClass {}

// ❌ Don't:
const something = require('./something');
module.exports = MyClass;
```

2. **TypeScript Configuration**
- Keep `strict: true` in tsconfig.json
- Use explicit type annotations for function parameters and returns
- Allow implicit types for variables when type can be inferred
```typescript
// ✅ Do:
function add(a: number, b: number): number {
  const result = a + b; // type inferred
  return result;
}

// ❌ Don't:
function add(a, b) {
  return a + b;
}
```

3. **Testing Setup**
- Use Jest with TypeScript via `ts-jest`
- Keep test files in `__tests__` directories
- Use `.test.ts` extension for test files
- Mock external dependencies explicitly
```typescript
// ✅ Do:
import { MyService } from '../services';
jest.mock('../services');

// ❌ Don't:
const MyService = jest.requireActual('../services');
```

4. **File Organization**
```
src/
  client/
    __tests__/        # Client-side tests
    components/
    scenes/
  server/
    __tests__/        # Server-side tests
    routes/
    services/
```

5. **Jest Configuration**
```typescript
{
  preset: 'ts-jest',
  testEnvironment: 'node',
  projects: [
    {
      displayName: 'server',
      testMatch: ['<rootDir>/src/server/__tests__/**/*.test.ts'],
      moduleFileExtensions: ['ts', 'js'],
      transform: {
        '^.+\\.ts$': 'ts-jest'
      }
    },
    {
      displayName: 'client',
      testMatch: ['<rootDir>/src/client/__tests__/**/*.test.ts'],
      testEnvironment: 'jsdom',
      setupFilesAfterEnv: ['<rootDir>/src/client/__tests__/setupTests.ts']
    }
  ]
}
```

6. **Test File Structure**
```typescript
import { Something } from '../path';

// Mock setup at top of file
jest.mock('../path/to/dependency');

describe('Component/Feature name', () => {
  let instance: Something;
  
  beforeEach(() => {
    instance = new Something();
  });

  afterEach(() => {
    jest.clearAllMocks();
  });

  it('should do something', () => {
    // Arrange
    const input = 'test';
    
    // Act
    const result = instance.doSomething(input);
    
    // Assert
    expect(result).toBe('expected');
  });
});
```

7. **Error Handling**
- Use typed errors
- Always catch and handle errors appropriately
- Provide meaningful error messages
```typescript
// ✅ Do:
try {
  await service.doSomething();
} catch (error) {
  if (error instanceof ServiceError) {
    // Handle specific error
  }
  throw error;
}

// ❌ Don't:
try {
  await service.doSomething();
} catch (e) {
  console.log(e);
}
```

8. **Async Code**
- Use async/await over raw promises
- Handle promise rejections
- Maintain proper error propagation
```typescript
// ✅ Do:
async function getData(): Promise<Data> {
  const result = await fetch('/api/data');
  return result.json();
}

// ❌ Don't:
function getData() {
  return fetch('/api/data')
    .then(res => res.json());
}
```

# 🔄 Real-Time State Synchronization: Avoiding Stale References

When working with real-time state (socket.io patches, WebSocket updates, polling), object references captured before async operations can become stale. This is a critical source of bugs in this codebase.

## The Problem

Socket patches can replace objects in arrays during async operations:

```typescript
// ❌ BUG: Stale reference pattern
async function handleAction() {
  // 1. Capture reference BEFORE async call
  const player = gameState.players[gameState.currentPlayerIndex];

  // 2. Async operation (server call, await, etc.)
  await api.doSomething();  // Socket patch may arrive here!

  // 3. Reference is now STALE - socket patch replaced the object in array
  player.someProperty = newValue;  // Updates OLD object, not the one in array!
}
```

## The Fix: Re-fetch After Async Operations

Always re-fetch object references from their source after any async operation:

```typescript
// ✅ CORRECT: Re-fetch after async
async function handleAction() {
  // 1. Capture reference for initial checks
  const player = gameState.players[gameState.currentPlayerIndex];
  const playerId = player.id;  // IDs are immutable, safe to keep

  // 2. Async operation
  await api.doSomething();

  // 3. Re-fetch the reference after async completes
  const refreshedPlayer = gameState.players[gameState.currentPlayerIndex];
  if (!refreshedPlayer || refreshedPlayer.id !== playerId) {
    console.error("Player reference became invalid");
    return;
  }

  // 4. Use the refreshed reference for mutations
  refreshedPlayer.someProperty = newValue;
}
```

## When This Applies

Re-fetch references after ANY of these operations:
- `await` on API calls (fetch, axios, service methods)
- `await` on any async function that may trigger socket events
- After `setTimeout`/`setInterval` callbacks
- After event handler completions that involve async work

## Safe Properties (No Re-fetch Needed)

These are safe to capture once and use throughout:
- **IDs**: `player.id`, `gameState.id` - immutable identifiers
- **Primitive values used for comparison**: values captured for validation
- **References to objects that are never replaced**: singletons, services

## Unsafe Properties (Must Re-fetch)

These require re-fetching after async operations:
- **Array elements**: `gameState.players[index]`, `state.items[i]`
- **Nested mutable objects**: `player.trainState`, `player.hand`
- **Any object that socket patches may replace**

## Common Bug Patterns in This Codebase

### Pattern 1: Movement History Not Persisting
```typescript
// ❌ BUG: movementHistory pushed to stale reference
const currentPlayer = gameState.players[index];
await moveTrainWithFees(position, gameId);  // Socket patch replaces player
currentPlayer.trainState.movementHistory.push(segment);  // Lost!

// ✅ FIX:
const currentPlayer = gameState.players[index];
const playerId = currentPlayer.id;
await moveTrainWithFees(position, gameId);
const refreshedPlayer = gameState.players[index];
if (refreshedPlayer?.id === playerId) {
  refreshedPlayer.trainState.movementHistory.push(segment);
}
```

### Pattern 2: Undo Not Restoring State
```typescript
// ❌ BUG: Restoring state to stale reference
const player = gameState.players.find(p => p.id === playerId);
await playerStateService.undoLastAction(gameId);  // Socket patch!
player.trainState.remainingMovement = previousMovement;  // Updates stale object

// ✅ FIX:
const playerBefore = gameState.players.find(p => p.id === playerId);
await playerStateService.undoLastAction(gameId);
const player = gameState.players.find(p => p.id === playerId);  // Re-fetch!
player.trainState.remainingMovement = previousMovement;
```

### Pattern 3: Hand/Inventory Not Updating
```typescript
// ❌ BUG: Using stale hand data
const existingPlayer = gameState.players[index];
const preservedHand = existingPlayer.hand;  // Captured before delivery
await deliverLoad(...);  // Hand is updated during this call
// Later: socket patch uses preservedHand which is outdated

// ✅ FIX: Get from authoritative source
const currentLocalPlayer = playerStateService.getLocalPlayer();
const preservedHand = currentLocalPlayer?.hand || existingPlayer.hand;
```

## Implementation Checklist

When writing async code that modifies game state:

1. ✅ Identify all object references captured before async operations
2. ✅ Determine which references may be replaced by socket patches
3. ✅ Re-fetch those references immediately after the async operation
4. ✅ Validate the refreshed reference (check ID matches, object exists)
5. ✅ Handle the case where the reference is no longer valid
6. ✅ Use refreshed references for all subsequent mutations

## Testing Considerations

Stale reference bugs are hard to reproduce in unit tests because:
- Tests don't have real socket connections
- Timing of patches is deterministic in mocks

To catch these bugs:
- Code review specifically for the stale reference pattern
- Integration tests with real socket connections
- Manual testing with network latency simulation

9. **Enum Guidelines**
- Use unique enum names across the entire codebase
- Never create enums with overlapping purposes (e.g., CityType vs TerrainType for cities)
- Document enum values with JSDoc comments
- Use PascalCase for enum names and values
- Prefix enum names with domain/category if needed to avoid conflicts
```typescript
// ✅ Do:
/** Represents all terrain types including cities */
export enum TerrainType {
  Clear = 0,
  Mountain = 1,
  MajorCity = 2,
  // ...
}

// ❌ Don't:
export enum TerrainType { /* ... */ }
export enum CityType { /* ... */ }  // Overlapping purpose

// ❌ Don't: Ambiguous naming
export enum Type { /* ... */ }
```

10. **Enum Value Management**
- Keep a central registry of all enum values to prevent duplicates
- Use numeric values explicitly to catch duplicate values at compile time
- Never mix string and numeric enum values within the same enum
```typescript
// ✅ Do:
export enum TerrainType {
  Clear = 0,
  Mountain = 1,
  MajorCity = 2
}

// ❌ Don't: Mixed value types
export enum TerrainType {
  Clear = 0,
  Mountain = "MOUNTAIN",
  MajorCity = 2
}

// ❌ Don't: Implicit values that could clash
export enum TerrainType {
  Clear,      // 0
  Mountain,   // 1
  MajorCity   // 2
}
```

# Type Changes Rule

When modifying types in the codebase:

1. Always check existing type definitions first before creating new ones
2. Only modify types when there is a clear requirement to do so such as missing property which must now go on the type or for design reasons you have been instructed to make a type change by the prompt.
3. When adding new functionality that uses existing types:
   - First try to use the existing type structure
   - Only extend types if the new functionality cannot be expressed with existing types
   - Document why the type extension is necessary
4. Never consolidate or split types without explicit requirements
5. When in doubt, ask for clarification before making type changes

# 🔄 Extending Existing Logic Rule

When adding new functionality to existing systems:

## Core Principles

### 1. **Extend, Don't Replace**
- Always look for opportunities to extend existing methods rather than creating new ones
- Reuse existing infrastructure and patterns
- Maintain consistency with existing architectural approaches

```typescript
// ✅ Do: Extend existing method
private async handleTrainPlacement(pointer: Phaser.Input.Pointer): Promise<void> {
  const nearestMilepost = this.findNearestMilepostOnOwnTrack(/* ... */);
  
  if (nearestMilepost) {
    if (this.isTrainMovementMode) {
      await this.handleMovement(currentPlayer, nearestMilepost, pointer);
    }
    // Extend with new functionality
    if (this.isCity(nearestMilepost) && this.isSamePoint(nearestMilepost, currentPlayer.trainState.position)) {
      await this.handleCityArrival(currentPlayer, nearestMilepost);
    }
  }
}

// ❌ Don't: Create separate handler
private async handleCityClick(pointer: Phaser.Input.Pointer): Promise<void> {
  // Duplicates grid point finding logic
  const clickedPoint = this.trackDrawingManager.getGridPointAtPosition(/* ... */);
  // ... duplicate logic
}
```

### 2. **Unify Processing Pipelines**
- Process all related interactions through the same pipeline
- Use conditional branching within existing methods rather than separate handlers
- Keep the main entry point as the single source of truth

```typescript
// ✅ Do: Unified processing
this.scene.input.on("pointerdown", async (pointer: Phaser.Input.Pointer) => {
  const nearestMilepost = this.findNearestMilepostOnOwnTrack(/* ... */);
  if (nearestMilepost) {
    // All interactions go through same pipeline
    if (this.isTrainMovementMode) { /* handle movement */ }
    if (this.isCity(nearestMilepost) && this.isSamePoint(/* ... */)) { /* handle city */ }
  }
});

// ❌ Don't: Separate handlers
this.scene.input.on("pointerdown", async (pointer: Phaser.Input.Pointer) => {
  if (this.isTrainMovementMode) { /* movement logic */ }
});
this.scene.input.on("pointerdown", async (pointer: Phaser.Input.Pointer) => {
  if (!this.isDrawingMode) { /* city logic */ }
});
```

### 3. **Create Focused Helper Methods**
- Extract simple, testable utility functions for common checks
- Keep helper methods focused on single responsibilities
- Make them reusable across the codebase

```typescript
// ✅ Do: Focused helpers
private isCity(gridPoint: GridPoint): boolean {
  return (
    gridPoint.terrain === TerrainType.MajorCity ||
    gridPoint.terrain === TerrainType.MediumCity ||
    gridPoint.terrain === TerrainType.SmallCity
  );
}

private isSamePoint(point1: Point | null, point2: Point | null): boolean {
  if (point1 && point2) {
    return point1.row === point2.row && point1.col === point2.col;
  }
  return false;
}

// ❌ Don't: Complex abstractions
private async shouldShowLoadDialog(player: Player, cityData: any): Promise<boolean> {
  // Unnecessary abstraction that duplicates existing logic
}
```

### 4. **Maintain Existing Patterns**
- Follow the established patterns in the codebase
- Use the same architectural approaches as existing similar functionality
- Don't introduce new patterns unless absolutely necessary

```typescript
// ✅ Do: Follow existing pattern
// Existing: handleTrainPlacement -> handleMovement -> handleCityArrival
// New: handleTrainPlacement -> handleMovement + handleCityArrival (same pattern)

// ❌ Don't: Create new patterns
// New: separate input handlers, different processing pipeline
```

### 5. **Reuse Existing Infrastructure**
- Leverage existing methods and utilities
- Don't recreate functionality that already exists
- Extend existing methods rather than creating parallel ones

```typescript
// ✅ Do: Reuse existing methods
const nearestMilepost = this.findNearestMilepostOnOwnTrack(/* ... */);
await this.handleCityArrival(currentPlayer, nearestMilepost); // Existing method

// ❌ Don't: Recreate existing logic
const clickedPoint = this.trackDrawingManager.getGridPointAtPosition(/* ... */);
// ... recreate city arrival logic
```

## Implementation Checklist

When extending existing logic:

1. **Analyze the existing flow first** - understand how similar functionality is currently implemented
2. **Look for extension points** - identify where new functionality can be added to existing methods
3. **Reuse existing infrastructure** - leverage existing methods, utilities, and patterns
4. **Create focused helpers** - extract simple utility methods for common checks
5. **Maintain consistency** - follow the same patterns and architectural approaches
6. **Minimize changes** - add the minimum code necessary to achieve the goal
7. **Test the integration** - ensure the new functionality works with existing systems

## Common Anti-Patterns to Avoid

- **Creating separate handlers** for related functionality
- **Duplicating existing logic** instead of reusing it
- **Adding unnecessary abstractions** that don't provide value
- **Violating single responsibility** by creating complex multi-purpose methods
- **Breaking existing patterns** without good reason
- **Over-engineering solutions** when simple extensions would suffice

# ⚙️ TypeScript Game Architecture Guide for Coding Agent

## 🌟 Purpose

This guide outlines architectural principles and patterns for building a maintainable, scalable, and performant game in TypeScript. The target game uses a hex/grid map, track building, object manipulation, and turn-based player logic.

---

## 📦 Core Architectural Principles

### 1. **Model Domain Objects as Classes or Interfaces**

* Every core game concept (e.g., `Train`, `City`, `LoadChip`, `TrackSegment`) should be a distinct class or interface.
* Encapsulate data and methods within each object:

  ```ts
  class Train {
    type: TrainType;
    position: GridPoint;
    draw(): void;
    move(path: GridPoint[]): void;
  }
  ```

---

### 2. **Separate Game Logic from Rendering**

* Keep rendering code in `View` or `Sprite` classes.
* Logic classes (`Train`, `Track`, `GameState`) **must not access rendering APIs** (e.g., Phaser).
* Use composition:

  ```ts
  class TrainView {
    constructor(public train: Train, public graphics: Graphics) {}
    draw(): void { ... }
  }
  ```

---

### 3. **Centralized Game State**

* Use a single `GameState` structure to manage game data:

  ```ts
  interface GameState {
    players: Player[];
    trains: Record<string, Train>;
    board: GridMap;
    currentPlayerId: string;
    turnPhase: TurnPhase;
  }
  ```

---

### 4. **Strong Typing & Enum Modeling**

* Use `type` aliases and `enum`s for clear expression of rules and constraints:

  ```ts
  type TrainType = 'Freight' | 'FastFreight' | 'HeavyFreight' | 'Superfreight';
  enum TerrainType { Clear, Mountain, Alpine, River, Ocean }
  ```

---

### 5. **Use Grid Utilities for Map Logic**

* Movement and neighbor logic should be encapsulated in a `Grid` class:

  ```ts
  class HexGrid {
    getNeighbors(point: GridPoint): GridPoint[];
    distance(a: GridPoint, b: GridPoint): number;
  }
  ```

---

## 🧠 Behavioral Modeling

### 6. **Use State Machines for Turn Phases**

* Each player’s turn follows a strict sequence:

  * Move train
  * Load/unload
  * Deliver
  * Build/upgrade
* Represent this using a state machine or turn-phase enum.

---

### 7. **Encapsulate Drawing Behavior per Object Type**

* Prefer putting drawing logic inside each model or associate it closely with it:

  * `Train.draw(graphics)`
  * `City.draw()`
  * Avoid large procedural rendering functions.

---

### 8. **Event-Driven Updates**

* Use an event bus or observer pattern to signal game state changes:

  ```ts
  eventBus.emit('loadPickedUp', { trainId, loadType });
  ```

---

### 9. **Avoid Global State Outside Game Context**

* Do not rely on global variables.
* Use dependency injection for services such as sound, rendering layers, or network.

---

## 🧪 Testability and Maintainability

### 10. **Pure Logic Should Be Easily Unit-Tested**

* Ensure pathfinding, rule enforcement, and game rules are testable without graphics or UI.

### 11. **Use Factory Functions for Complex Creation**

* Example: `createTrain(trainType: TrainType, startPoint: GridPoint): Train`

---

## 🛠️ Suggested Folder Structure

```
/models        # Domain logic (Train, City, Load, Track)
/views         # Rendering code and sprite helpers
/state         # Game state container and reducers
/utils         # Grid math, helpers
/engine        # Turn manager, event dispatcher, AI agent
/config        # Static data (terrain costs, load types)
```

---

## 📌 Summary Checklist

| ✅ Best Practice               | 💬 Summary                                     |
| ----------------------------- | ---------------------------------------------- |
| Domain models encapsulated    | `Train`, `City`, `Load` are self-contained     |
| Rendering isolated from logic | Views handle Phaser; logic is UI-agnostic      |
| Global state centralized      | Use `GameState` or reducer pattern             |
| Typed enums for logic safety  | Use `enum` and `type` aliases                  |
| Grid logic abstracted         | `HexGrid.getNeighbors`, `GridUtils.snapToGrid` |
| Events not direct callbacks   | Use event bus for gameplay updates             |
| Turn sequence enforceable     | Represented explicitly via state machine       |
| View class per game object    | Avoid centralized `drawAll` logic              |
| Test coverage on pure logic   | Model rules testable in isolation              |


1. Per-Test Cleanup, Not Global Truncation
Always use per-fixture (per-test or per-suite) cleanup with explicit DELETE FROM statements in dependency order (child tables first, then parent tables).
Never use global TRUNCATE or disable triggers for test cleanup unless absolutely necessary and safe (e.g., no open transactions, single-threaded).
2. No Cleanup During Transactions
Never perform database cleanup (truncate, delete, or schema changes) while any transaction is open.
Always ensure all transactions are committed or rolled back before cleanup.
3. Minimal State Setup
Each test must set up only the state it needs.
Never rely on state from previous tests or global fixtures.
4. Test Independence
Tests must not depend on the order of execution or on side effects from other tests.
Each test should be able to run in isolation and still pass.
5. Dependency Order for Deletes
When deleting rows for cleanup, always delete from child tables before parent tables to avoid foreign key constraint errors and deadlocks.
6. Avoid Shared Clients in Tests
Prefer using a fresh database client per test or per suite, unless there is a strong reason to share.
Always release clients after use.
7. Serial Execution for DB-Heavy Tests
If tests interact heavily with the database and are not designed for parallelism, run them serially (one after another) to avoid race conditions and deadlocks.
8. No Disabling of Triggers Unless Absolutely Necessary
Disabling triggers can break referential integrity and should be avoided in test cleanup.
9. Explicit Error Handling
Always check for and handle errors during setup and cleanup, and fail tests clearly if cleanup/setup fails.
10. Document Test Cleanup Strategy
Always document the cleanup and isolation strategy at the top of each test file for future maintainers.
Disabling triggers can break referential integrity and should be avoided in test cleanup.
9. Explicit Error Handling
Always check for and handle errors during setup and cleanup, and fail tests clearly if cleanup/setup fails.
10. Document Test Cleanup Strategy
Always document the cleanup and isolation strategy at the top of each test file for future maintainers.

---
> Source: [jeffgabriel/eurorails_ai](https://github.com/jeffgabriel/eurorails_ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
