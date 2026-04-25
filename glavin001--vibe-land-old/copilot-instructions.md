## vibe-land-old

> Global rules always followed

# Vibe Coding Starter Pack: 3D Multiplayer - Technical Guide

This document describes the architecture, setup, workflow, and key development patterns for the Vibe Coding Starter Pack 3D Multiplayer project. **SpacetimeDB v2.0.1** is used throughout.

## 1. Project Overview

*   **Goal:** Provide a foundation for building 3D multiplayer web games using SpacetimeDB as the backend.
*   **Backend:** SpacetimeDB module written in Rust (`server/`). Handles game logic, state management, and data persistence. Compiled to WASM.
*   **Frontend:** Client application built with Vite, React 19, TypeScript, and React Three Fiber (`client/`). Connects to SpacetimeDB via WebSocket, subscribes to data, calls reducers, and renders the game state.

## 2. Game Architecture

The starter pack uses a modular, component-based architecture with the following core systems:

1. **Core Game Loop**: Managed in `client/src/App.tsx`, which initializes all components and runs the input send cycle at 20Hz
2. **Rendering System**: Handled by Three.js via React Three Fiber with scene setup in `client/src/components/GameScene.tsx`
3. **Character System**: Player controls, FBX model loading, animations, and client-side prediction in `client/src/components/Player.tsx`
4. **UI System**: HUD elements and feedback in `client/src/components/PlayerUI.tsx` and `client/src/components/DebugPanel.tsx`
5. **Multiplayer System**: Client-server communication in `client/src/App.tsx` and server-side code in `server/src/` directory
6. **Load Testing**: Bot simulation in `client/src/simulation.ts` for stress-testing with configurable bot count

## 3. Core SpacetimeDB v2 Concepts

*   **Tables:** Relational data storage defined as Rust structs with `#[spacetimedb::table]`. Key tables include `player` and `logged_out_player`. Tables must be marked `public` for client access. The `name = "..."` attribute defines the SQL table name; the `accessor = "..."` attribute defines the Rust accessor name used in `ctx.db`. Table names in client SQL subscriptions are **case-sensitive** (e.g., `name = "player"` requires `SELECT * FROM player`).
*   **Reducers:** Atomic, transactional Rust functions (`#[spacetimedb::reducer]`) that modify table state. Triggered by client calls or internal events. Key reducers: `register_player`, `update_player_input`, lifecycle reducers (`identity_connected`, `identity_disconnected`).
*   **Subscriptions:** Clients subscribe to SQL queries (e.g., `SELECT * FROM player`) using `conn.subscriptionBuilder()`. **IMPORTANT:** Register `.onApplied()` and `.onError()` callbacks *before* calling `.subscribe()` — in v2, backfill data fires immediately on subscribe.
*   **Generated Bindings:** The `spacetime generate` command creates TypeScript code (`client/src/generated/`) based on the Rust module's schema, providing type-safe access to tables, reducers, and types on the client. Types are in `generated/types.ts`.
*   **Identity:** Represents a unique, authenticated user. In v2 reducers: `ctx.sender()` (method call with parentheses). On client: `conn.identity`. Used as the primary key for player-related tables.

## 4. Prerequisites

1.  **Rust & Cargo:** ([https://rustup.rs/](mdc:https:/rustup.rs))
    *   Version: **1.93+** required for SpacetimeDB 2.0.1
    *   Install: `curl https://sh.rustup.rs -sSf | sh`
    *   Add WASM target: `rustup target add wasm32-unknown-unknown`
    *   Ensure `~/.cargo/bin` is in PATH (restart terminal or `source ~/.cargo/env`).
2.  **SpacetimeDB CLI (v2.0.1):** ([https://install.spacetimedb.com](mdc:https:/install.spacetimedb.com))
    *   Install: `curl -sSf https://install.spacetimedb.com | sh`
    *   Ensure install location (e.g., `~/.local/bin`) is in PATH (restart terminal or `source ~/.zshrc`/`.bashrc`).
    *   Verify: `spacetime version` (Should show `2.0.1` or compatible).
3.  **Node.js 22+ & npm:** Install via [nvm](mdc:https:/github.com/nvm-sh/nvm) (`nvm install 22`).
4.  **(Optional) `wasm-opt`:** For optimizing the built Rust module (part of `binaryen`). Install via system package manager (e.g., `brew install binaryen`, `apt install binaryen`). If missing, builds will still work but show a warning.

## 5. Project Structure

```
vibe-coding-starter-pack-3d-multiplayer/
├── client/                   # Vite+React+TS client
│   ├── public/
│   │   └── models/           # FBX character models (wizard/, paladin/)
│   ├── src/
│   │   ├── components/       # GameScene, Player, DebugPanel, PlayerUI, JoinGameDialog
│   │   ├── generated/        # Auto-generated TS bindings (DO NOT EDIT)
│   │   ├── App.css           # Component styles
│   │   ├── App.tsx           # Main React component — connection, input, game loop
│   │   ├── simulation.ts     # Bot load-testing tool
│   │   ├── index.css         # Global styles/resets
│   │   └── main.tsx          # React entry point
│   ├── index.html
│   ├── package.json
│   ├── tsconfig.json         # Main TS config
│   ├── tsconfig.app.json     # App-specific TS config (used by Vite)
│   └── vite.config.ts
├── server/                   # SpacetimeDB Server Module (Rust)
│   ├── src/
│   │   ├── common.rs         # Shared types (Vector3, InputState), constants
│   │   ├── player_logic.rs   # Server-side movement calculation
│   │   └── lib.rs            # Tables, reducers, lifecycle handlers
│   ├── Cargo.toml            # spacetimedb = "2.0.1"
│   └── target/
├── CLAUDE.md                 # AI assistant project context
├── setup.sh                  # One-command setup script
└── README.md                 # Project overview
```

## 6. Development Workflow

**Three terminals are needed.**

**Terminal 1: SpacetimeDB Server**

1.  **Navigate:** `cd server`
2.  **(If Rust code changed) Build Module:**
    ```bash
    spacetime build
    ```
3.  **Start Server:**
    ```bash
    spacetime start
    ```
    *(Leave this running)*

**Terminal 2: Publish & Manage**

1.  **Navigate:** `cd server`
2.  **(First run or after code changes) Publish Module:**
    ```bash
    spacetime publish vibe-land
    # Use -y flag for major version upgrades:
    spacetime publish vibe-land -y
    ```
    *CRITICAL: This uploads your built module to the **running** server instance.*
3.  **(If schema/reducers changed) Regenerate Bindings:**
    ```bash
    spacetime generate --lang typescript --out-dir ../client/src/generated
    ```

**Terminal 3: Vite Client**

1.  **Navigate:** `cd client`
2.  **(If dependencies changed) Install Dependencies:**
    ```bash
    npm install
    ```
3.  **Run Dev Server:**
    ```bash
    npm run dev
    ```
    *Access the client at http://localhost:5173. Check browser console for logs.*

**Load Testing:**
```bash
cd client
npm run simulate              # 10 bots, 10 seconds
npm run simulate -- 100 30    # 100 bots, 30 seconds
```

**Stopping:** Use `Ctrl+C` in each terminal.

## 7. Server (Rust Module Details)

*   **Structure:** Logic is modularized (`player_logic.rs`, `common.rs`). `lib.rs` defines schema and reducers, calling functions in logic modules.
*   **Schema (`lib.rs`):** Use `#[spacetimedb::table(name = "snake_case", public)]`. The `accessor = "..."` attribute defines the Rust accessor name for `ctx.db`. Add `#[derive(Clone)]`. Use `Identity` for player PKs.
*   **Reducers (`lib.rs`):** Use `#[spacetimedb::reducer]`. Lifecycle reducers (`init`, `client_connected`, `client_disconnected`) take **only** `ctx: &ReducerContext`.
*   **DB Access (Within Modules):**
    *   Import traits: `use crate::player;`
    *   Import data structs: `use crate::PlayerData;`
    *   Access via context: `ctx.db.accessor_name()`
    *   Methods: `.identity().find(id)`, `.identity().update(data)`, `.identity().delete(id)`, `.insert(data)`.
*   **Sender:** `ctx.sender()` — **method call** in v2 (returns `Identity`). NOT a field.
*   **Timestamp:** `ctx.timestamp` — **field** access (NOT a method). Provides a `Timestamp` struct (u64 microseconds). Access value with `.micros()`.
*   **Error Handling:** Return `Result<(), String>` for handled errors. Panics also roll back transactions.
*   **Constants (`common.rs`):** `PLAYER_SPEED = 7.5`, `SPRINT_MULTIPLIER = 1.8`. Client must match these values for accurate prediction.

## 8. Client (React+TS+R3F Details)

*   **Setup:** Use Vite `react-ts` template. Install `spacetimedb@^2.0.0` (NOT `@clockworklabs/spacetimedb-sdk`), `three`, `@react-three/fiber`, `@react-three/drei`, `@types/three`.
*   **Bindings:** Import directly from generated modules:
    ```typescript
    import { DbConnection, EventContext, ErrorContext } from './generated';
    import { PlayerData, InputState } from './generated/types';
    ```
*   **Connection:** Use `DbConnection.builder()...build();` pattern in a top-level `useEffect(..., [])` hook in `App.tsx`:
    ```typescript
    DbConnection.builder()
      .withUri('ws://localhost:3000')
      .withDatabaseName('vibe-land')   // NOT withModuleName
      .withConfirmedReads(false)              // Reduces latency (v2 defaults to confirmed reads)
      .onConnect(onConnect)
      .onDisconnect(onDisconnect)
      .build();
    ```
*   **Connection Object (`conn`):** The `DbConnection` instance received in the `onConnect` callback is essential. Store it (e.g., in a module-level variable) to access `db` and `reducers`.
*   **Reducer Calls:** Use `conn.reducers.reducerName({ arg1, arg2 })` (camelCase, **single object argument** in v2). Ensure `conn` is not null.
    ```typescript
    conn.reducers.registerPlayer({ username, characterClass });
    conn.reducers.updatePlayerInput({ input, clientPos, clientRot, clientAnimation });
    ```
*   **Table Access:** `conn.db.tableName` is **property access** (no parentheses). NOT `conn.db.tableName()`.
    ```typescript
    conn.db.player.onInsert(callback);   // correct
    conn.db.player.iter();               // correct
    ```
*   **Table Callbacks:** Register *after* connection but **BEFORE subscribing** — in v2, backfill fires immediately on subscribe. Callbacks receive `(ctx: EventContext, row: RowType, ...)`.
*   **State Management:** Use React `useState` for data received from SpacetimeDB (e.g., `players` map). Update state within table callbacks (`setPlayers(...)`). Render components based on this state.
*   **Stale Closure Prevention:** Use `useRef` for values accessed in callbacks (identity, localPlayer, connected status). React `useState` values captured in `useCallback` go stale. See `identityRef`, `localPlayerRef`, `connectedRef` in App.tsx.
*   **Game Loop Stability:** Use `sendInputRef` pattern — wrap `sendInput` in a ref so the game loop `useEffect` doesn't restart on every state change.
*   **Identity Comparison:** Use `.toHexString()` for reliable comparison and map keys.
*   **Subscriptions:** Use `conn.subscriptionBuilder()`. Register `.onApplied()` and `.onError()` callbacks **before** calling `.subscribe("SELECT * FROM table_name")` (snake_case, case-sensitive).
*   **JSX/TS Config:** Ensure `tsconfig.app.json` (or `tsconfig.json`) has `compilerOptions.jsx` set to `"react-jsx"`. Files with JSX need the `.tsx` extension.

## 9. Multiplayer Architecture

The game implements a client-server architecture for multiplayer functionality:

1. **Server**: SpacetimeDB Rust module compiled to WebAssembly
   - Server-authoritative game state
   - Handles player connections, movement, and interactions
   - Manages game state via tables
   - Broadcasts state updates to all subscribed clients automatically

2. **Client**: React Three Fiber client with SpacetimeDB connection
   - Sends player inputs to server at 20Hz (50ms intervals)
   - Receives and applies state updates from server via subscriptions
   - Uses client-side prediction for responsive movement (actual frame `dt` from `useFrame`, NOT a fixed delta)
   - Renders both local player (with camera follow) and remote players

3. **Communication Protocol**:
   - WebSocket connection to `ws://localhost:3000`
   - Subscription-based state updates (`SELECT * FROM player`)
   - Input events sent via `updatePlayerInput` reducer

4. **Server Components**:
   - `server/src/lib.rs`: Tables, reducers, lifecycle handlers
   - `server/src/player_logic.rs`: Server-side movement calculation (`delta_time = 1/20`)
   - `server/src/common.rs`: Shared types and constants

5. **Client Multiplayer Components**:
   - `client/src/App.tsx`: Connection, input handling, game loop
   - `client/src/components/Player.tsx`: Character rendering, animation, prediction
   - `client/src/components/DebugPanel.tsx`: Connected player list and game state

## 10. Key Learnings & Gotchas

*   **`spacetime publish` is MANDATORY:** Run *after* `build` and *while* `start` is running. Updates module code on the server instance. Use `-y` flag for major version upgrades.
*   **Version Alignment:** Client SDK `spacetimedb@^2.0.0` in `package.json`, server SDK `spacetimedb = "2.0.1"` in `Cargo.toml`, CLI v2.0.1. These must be compatible.
*   **Package Rename:** The npm package is `spacetimedb` (NOT `@clockworklabs/spacetimedb-sdk`). The old package was for v1.
*   **`ctx.sender()` vs `ctx.timestamp`:** In v2, `ctx.sender()` is a **method** (with parentheses). `ctx.timestamp` is still a **field** (no parentheses). Getting these wrong causes compilation errors.
*   **Reducer Arguments:** v2 reducers take a **single object argument** on the client side, not positional args. E.g., `registerPlayer({ username, characterClass })` not `registerPlayer(username, characterClass)`.
*   **`withConfirmedReads(false)`:** v2 defaults to confirmed reads (waits for durability), which adds latency. Always add this to the connection builder for games.
*   **Register Callbacks Before Subscribing:** In v2, backfill data fires immediately on `.subscribe()`. If callbacks aren't registered yet, you miss the initial data load.
*   **Stale React Closures:** Use `useRef` for identity, localPlayer, and connected status. Values captured in `useCallback`/`useEffect` closures at creation time go stale when state updates.
*   **Run Commands in Correct Directory:** `spacetime` commands usually in `server`, `npm` commands usually in `client`.
*   **Table Visibility:** Forget `public` on Rust tables? Client subscriptions will fail (`'table_name' is not a valid table`).
*   **Schema Changes:** Adding/removing columns requires `spacetime delete <db_name>` then `spacetime publish <db_name>` during local dev.
*   **Case Sensitivity:** Be precise with table names in SQL (`Player` vs `player`) and reducer names in client calls (`registerPlayer` vs `register_player`). Follow Rust definitions for SQL, camelCase for client reducer calls.
*   **React StrictMode:** Can cause double `useEffect` runs in dev mode. Remove `<React.StrictMode>` wrapper in `main.tsx` if encountering double connections/prompts.
*   **Client-Side Prediction:** Must use actual frame `dt` from `useFrame`, not a fixed `SERVER_TICK_DELTA`. Speed constants (`PLAYER_SPEED = 7.5`) must match server values in `common.rs`.
*   **Client State:** Render UI based on React state variables (`players`, `localPlayer`) updated by SpacetimeDB callbacks, not by directly iterating the potentially stale client cache (`conn.db.player.iter()`) during render.
*   **Load Testing:** Use `npm run simulate -- [numBots] [durationSeconds]` from the `client/` directory to spawn bot clients. Open the browser client simultaneously to see bots in the 3D scene.

## 11. Code Organization Best Practices

1. **Component Structure**:
   - Keep related functionality together
   - Separate visual elements from game logic
   - Use event-based communication between systems
   - Divide client and server responsibilities clearly

2. **Performance**:
   - Optimize render loops with proper delta time calculations
   - Reuse geometries and materials when possible
   - Use object pooling for frequently created/destroyed objects
   - Implement level-of-detail for complex objects
   - Optimize network traffic with relevance filtering

3. **Code Structure**:
   - Use TypeScript types for game entities
   - Use module imports rather than global variables
   - Keep methods focused on single responsibilities
   - Clearly separate client-side and server-side logic

4. **Animation**:
   - Use FBX models for character animations (Wizard scale: 0.02, Paladin scale: 1.0)
   - Implement proper animation transitions
   - Handle animation synchronization between server and client
   - Models live in `client/public/models/wizard/` and `client/public/models/paladin/`

5. **Multiplayer Guidelines**:
   - Always consider authority and validation for all game actions
   - Implement client-side prediction for responsive controls
   - Use interpolation for smooth remote player movement
   - Handle disconnections and reconnections gracefully

## 12. Expansion Guidelines

When adding new features:
1. Follow the existing component architecture
2. Create new files for major new systems
3. Extend existing classes rather than modifying them
4. Add appropriate event emissions for cross-component communication
5. Document new systems with clear comments
6. Maintain consistent naming conventions
7. Consider performance implications, especially for visual effects
8. For multiplayer features, always consider server authority, validation, and state synchronization

## 13. Development Process for New Features

1. **Planning Phase**:
   - Scan all relevant files the feature may impact
   - Identify any dependencies
   - Map out technical requirements in detail
   - Create a detailed markdown tasklist

2. **Implementation Phase**:
   - Implement server-side logic in Rust
   - Build and publish: `spacetime build && spacetime publish vibe-land -y`
   - Generate updated TypeScript bindings: `spacetime generate --lang typescript --out-dir ../client/src/generated`
   - Implement client-side components
   - Test with multiple clients and/or simulation bots

3. **Testing**:
   - Test with multiple simultaneous connections (use `npm run simulate`)
   - Verify behavior with high latency
   - Check error conditions and edge cases
   - Open browser client while bots are running to visually verify

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Glavin001) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
