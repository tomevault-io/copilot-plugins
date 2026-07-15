## architecture

> determining new files to create, how to associate new code with exsting code, making big changes

# EuroRails AI Architecture

## Overview

EuroRails AI is a multi-player railroad building game built with Phaser.js and Node.js. The architecture emphasizes clear separation between shared game state and per-player operations to support turn-based multiplayer gameplay.

**Related Documentation**:
- [TURN_MANAGEMENT_PLAN.md](./TURN_MANAGEMENT_PLAN.md) - Multi-player turn management implementation plan
- [TURN_MANAGEMENT_ARCHITECTURE_REVIEW.md](./TURN_MANAGEMENT_ARCHITECTURE_REVIEW.md) - Detailed architecture review
- [PLAYER_STATE_IMPLEMENTATION_SUMMARY.md](./PLAYER_STATE_IMPLEMENTATION_SUMMARY.md) - Implementation summary

## Component Architecture

The EuroRails AI game follows a component-based architecture to manage complexity and separate concerns. The main components are:

### Client-Side Components

1. **Scenes**
   - `GameScene`: Main game scene, orchestrates all other components
   - `SetupScene`: Handles game creation and player setup
   - `SettingsScene`: Manages game settings and player editing

2. **Components**
   - `MapRenderer`: Manages the game grid, terrain rendering, and point calculations
   - `TrackDrawingManager`: Handles track building, validation, and visualization
   - `UIManager`: Manages UI elements such as player info and controls
   - `CameraController`: Handles camera movement, zooming, and state persistence

3. **Services**
   - `GameStateService`: Manages **shared game state** (turn management, game status, read-only access)
   - `PlayerStateService`: Manages **local player operations** (money, position, loads, demand cards)
   - `LoadService`: Manages load availability and distribution
   - `TrackService`: Manages track network storage and retrieval

### Server-Side Services

1. **Database Services**
   - `playerService`: Player CRUD operations and authentication
   - `trackService`: Track network storage and retrieval
   - `gameService`: Game state management and camera state
   - `demandDeckService`: Demand card deck management

2. **Shared Services**
   - `TrackNetworkService`: Graph representation and pathfinding algorithms
   - `TrackBuildingService`: Track building validation and cost calculation
   - `IdService`: ID generation for game entities

## Multi-Player Architecture

### State Separation

The architecture separates **shared game state** from **per-player state**:

**Shared State** (GameStateService):
- Turn management (`currentPlayerIndex`)
- Game status (setup, active, completed)
- Read-only player list access
- Global game settings

**Per-Player State** (PlayerStateService):
- Local player identification via `userId`
- Money, position, loads management
- Demand card operations
- Player-specific updates

**Visual State** (All players see):
- Track networks (via TrackService)
- Player positions on map
- Train sprites
- Load availability in cities

**Private State** (Local player only):
- Demand cards in hand
- Camera view state (per-player in Phase 2)
- Local game view preferences

### Local Player Identification

Each client identifies its local player through:
1. **Primary**: Match `userId` from localStorage auth against players
2. **Fallback**: Single-player games use first player as local player
3. **Security**: All state updates validate local player identity

## Data Flow

1. User interacts with the UI (managed by `UIManager`)
2. Events are routed to appropriate components:
   - Map interactions → `MapRenderer`
   - Track building → `TrackDrawingManager`
   - Camera control → `CameraController`
3. State changes are processed by appropriate service:
   - **Shared state** (turns, status) → `GameStateService`
   - **Local player actions** → `PlayerStateService`
   - **Track operations** → `TrackService`
4. State is persisted to the server via API calls
5. Updates are rendered by the respective components
6. **Multi-player sync**: Shared state changes propagate to all clients

## Design Decisions

- **Component Separation**: Each component has a specific responsibility
- **Stateful Services**: Services maintain their own state to reduce coupling
- **Event-Driven Communication**: Components communicate via callbacks and events
- **Client-Server Architecture**: Game state is validated both client-side and server-side
- **Type Safety**: Strong TypeScript typing throughout the codebase
- **Multi-Player State Separation**: Clear boundaries between shared and per-player state
- **Security by Design**: Local player validation ensures only authenticated player can modify their state
- **Backward Compatibility**: Fallback mechanisms support legacy games without userId

## Server-Authoritative State Management

EuroRails AI uses a **server-authoritative** state management pattern to ensure consistency across all clients and prevent state desynchronization issues.

### State Management Pattern

All game state updates must follow these rules:

#### 1. API First Principle
- **Always make API call first**, never update local state before API succeeds
- Local state should only be updated after receiving a successful response from the server
- This ensures the server is the single source of truth for game state

**Correct Pattern:**
```typescript
// ✅ Correct: API first, then update local state
const response = await api.updatePlayerMoney(gameId, newMoney);
if (response.ok) {
  const updatedPlayer = await response.json();
  this.localPlayer.money = updatedPlayer.money; // Update only after success
}
```

**Incorrect Pattern:**
```typescript
// ❌ Incorrect: Updating local state before API call
this.localPlayer.money = newMoney; // Don't do this!
await api.updatePlayerMoney(gameId, newMoney);
```

#### 2. Single Source of Truth
Each state domain has one authoritative source:

- **Game state**: `GameScene.gameState` (passed to `GameStateService`)
- **Player state**: Accessed via `GameStateService.getCurrentPlayer()` or `PlayerStateService.getLocalPlayer()`
- **Track state**: Managed by `TrackDrawingManager` with API persistence
- **Load state**: Managed by `LoadService` with API persistence

#### 3. Error Handling
- On API failure, show error to user, **do NOT update local state**
- Never assume an API call succeeded - always check the response
- If an API call fails, the local state should remain unchanged

**Example:**
```typescript
try {
  const response = await api.updatePlayerLoads(gameId, loads);
  if (!response.ok) {
    throw new Error('Failed to update loads');
  }
  // Only update local state after successful API call
  this.localPlayer.trainState.loads = loads;
} catch (error) {
  // Show error, but don't update local state
  console.error('Failed to update loads:', error);
  showErrorToUser('Failed to update loads. Please try again.');
}
```

#### 4. State Sync
After a successful API call:
- Update local state from the API response, OR
- Wait for socket update to sync state across all clients
- Do not update local state from both sources (avoid double updates)

#### 5. localStorage Usage Restrictions
- **localStorage is NOT used for in-game state**
- localStorage is only for:
  - Authentication tokens (`eurorails.jwt`)
  - User information (`eurorails.user`)
  - Lobby state (`eurorails.currentGame`, `eurorails.currentPlayers`) - saved after API calls, not before
- In-game state (player money, loads, position, etc.) must never be stored in localStorage

#### 6. Socket Updates
- All state changes are broadcast via socket updates from route handlers
- Clients update local state from socket patches
- Socket updates are the primary mechanism for multi-player state synchronization

### Socket Update Pattern

Real-time state synchronization uses a **handler-owned socket updates** pattern where each route handler is responsible for emitting socket updates for its domain.

#### Handler-Owned Socket Updates

Each route handler that modifies shared game state should:

1. **Process the request** - Validate input, authenticate user
2. **Update database** - Persist changes via service layer
3. **Emit socket update** - Broadcast state changes to all clients in the game
4. **Return response** - Send HTTP response to the requesting client

**Example Pattern:**
```typescript
router.post('/update', async (req, res) => {
  // 1. Process request
  const { gameId, player } = req.body;
  
  // 2. Update database
  await PlayerService.updatePlayer(gameId, player);
  
  // 3. Emit socket update
  const io = getSocketIO();
  if (io) {
    io.to(gameId).emit('state:patch', {
      patch: { players: [player] },
      serverSeq: Date.now()
    });
  }
  
  // 4. Return response
  return res.status(200).json({ message: 'Player updated successfully' });
});
```

#### Socket Event Format

All socket updates use a consistent format:

```typescript
{
  patch: Partial<GameState>,  // Only changed data, not full state
  serverSeq: number            // Sequence number for ordering
}
```

**Guidelines:**
- Include only changed data in the patch, not the full state
- Use `serverSeq` for ordering (can use timestamp initially)
- Patches are merged into existing state on the client side

#### When to Emit Socket Updates

Emit socket updates for:
- ✅ Player state changes (money, loads, position, train type)
- ✅ Turn changes
- ✅ Track building
- ✅ Load pickup/delivery/dropping
- ✅ Demand card fulfillment

Do NOT emit socket updates for:
- ❌ Per-player camera state (not shared)
- ❌ Read-only operations (GET requests)
- ❌ Authentication operations

#### Client Socket Handling

Clients receive socket updates and merge them into local state:

1. `game.store.ts` receives `state:patch` events via socket service
2. `applyStatePatch` merges the patch into the current game state
3. `GameScene` listens to state changes and refreshes `this.gameState`
4. Services update their local copies when socket updates arrive

This ensures all clients stay synchronized without polling.

## Future Improvements

- **Per-Player Camera State**: Each player maintains independent view settings
- **Turn-Based Interaction Controls**: UI elements enable/disable based on active player
- **Real-Time Multi-Player Sync**: Socket.IO integration for live updates (✅ Implemented)
- **Spectator Mode**: Allow viewing games without being a player
- **Load Balancing**: Optimize for games with 6+ simultaneous players
- **Performance**: Optimized rendering for large maps with many players
- **Testing**: Comprehensive unit and integration tests for multiplayer scenarios

---
> Source: [jeffgabriel/eurorails_ai](https://github.com/jeffgabriel/eurorails_ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
