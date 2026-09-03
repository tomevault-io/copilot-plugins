## cyber-elephant

> A real-time, web-based white elephant gift exchange game for the UpLevel Ops holiday party. Lightweight, fun, and polished. Rust backend, TS frontend, shared game logic.

# PROJECT.md – UpLevel White Elephant

## Project Name: **UpLevel White Elephant**

## One-Liner
A real-time, web-based white elephant gift exchange game for the UpLevel Ops holiday party. Lightweight, fun, and polished. Rust backend, TS frontend, shared game logic.

---

## Goals
- Deliver a single working internal game that feels clean, modern, and slick.
- Showcase UpLevel’s ability to build **custom interactive tools quickly** using Rust + modern web tech.
- Support <20 players simultaneously via WebSockets.
- Keep the scope tight and avoid unnecessary productization.

---

## Core Architecture

### **Backend: Rust (Axum)**
- REST API for:
  - Creating/joining the game
  - Submitting gift data
  - Starting the game
- WebSocket endpoint for:
  - Broadcasting authoritative game state
  - Receiving player actions
- Shared `game-core` crate with:
  - `Game`, `Player`, `Gift`, `Action`, `GamePhase`, `GameEvent`
  - Deterministic state transitions: `apply_action(game, action) -> Result<Vec<GameEvent>, GameError>`
- Server is the single source of truth for all game state.
- Simple Postgres or in-memory store; persistent storage optional.

---

## Frontend: React or SvelteKit + TypeScript + Tailwind
- SPA with 3 primary screens:
  1. **Lobby** – Join by name, see list of players.
  2. **Gift Submission** – Enter `product_url` + `hint`. Show gift preview.
  3. **Game Board** – Realtime UI for turns, opening gifts, stealing, animations.

- WebSocket client to:
  - Receive `GameState` updates
  - Send `PlayerAction` messages
- Tailwind for styling, with a clean, modern visual feel.
- UpLevel branding minimal but present.

---

## Game Rules (Classic White Elephant)
- 1 gift per player, submitted before game start.
- Turn order randomized at game start.
- On a player’s turn, they may:
  - **Choose** an unopened gift
  - **Steal** an already opened gift (if rules allow)
- A gift may be stolen up to **3 times total**.
- No immediate steal-back: a player cannot steal from the person who just stole from them.
- Forced steal chains:
  - When a player’s gift is stolen, *they* immediately take a turn until someone eventually opens a new gift.
- Game ends when all gifts are opened and every player holds exactly one gift.

---

## Data Models (High-Level)

### Player
```
id: string
name: string
joined_at: number
```

### Gift
```
id: string
submitted_by: string
product_url: string
hint: string
image_url?: string
title?: string
opened_by?: string
stolen_count: number
state: "unopened" | "opened"
```

### Game State
```
id: string
phase: "lobby" | "submissions" | "in_progress" | "finished"
players: Player[]
gifts: Gift[]
turn_order: string[]        // array of player IDs
current_turn: number        // index into turn_order
active_player: string       // player ID
history: GameEvent[]
```

### Player Actions (via WebSocket)
```
type PlayerAction =
  | { type: "choose_gift", gift_id: string }
  | { type: "steal_gift", gift_id: string }
```

### Game Events (broadcast to all clients)
```
type GameEvent =
  | { type: "gift_opened", player_id: string, gift_id: string }
  | { type: "gift_stolen", from: string, to: string, gift_id: string }
  | { type: "turn_changed", player_id: string }
  | { type: "game_finished" }
```

---

## Features Required for v1 (No More Than This)
- Players join via link + name.
- Players submit URL + hint and see preview.
- Host starts game; submissions lock.
- Turn-by-turn game flow enforced server-side.
- Gift reveal modal with image/title.
- “Steal” option when valid; disabled when invalid.
- Animations:
  - Gift opening (simple scale/opacity)
  - Gift stealing (wiggle or bounce)
- End screen with list: `player → gift → link`.
- Export/copy-to-clipboard for host.

---

## Specifically Out of Scope (Important)
- User accounts / auth.
- Multiple concurrent games.
- Payment flows.
- Shipping address collection.
- Admin dashboards.
- Persistence beyond the session.
- OAuth or external API integrations.
- Ecommerce integrations.

THIS IS PURELY AN INTERNAL HOLIDAY TOOL.

---

## Quality Bar / Vibe
- Smooth, modern, minimal UI.
- No jank on animations or turn flow.
- Game state always correct—even if UI glitches, the backend state rules.
- Easy to run locally (docker-compose or single binary + static frontend).
- Code should be clean and modular but not enterprise over-engineered.

---

## Development Flow with LLM
When generating code, follow this workflow:

1. Make small, incremental changes.
2. Keep Rust logic in `game-core` isolated and tested.
3. Backend serves both the WebSocket API and static frontend build.
4. Frontend status should update live on any `GameEvent`.
5. Prefer simplicity and clarity over patterns.

---

## Final Deliverable Definition
A working local deployment where:
- 10–20 people can join via shared URL,
- submit gifts,
- play the game in real-time,
- see animations,
- and export the final results.

If it works cleanly for the UpLevel Ops holiday party, it’s a success.

---
> Source: [jsfillman/cyber-elephant](https://github.com/jsfillman/cyber-elephant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
