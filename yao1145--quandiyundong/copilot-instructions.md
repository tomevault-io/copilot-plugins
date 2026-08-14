## quandiyundong

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

《圈地运动》(Enclosure Movement) — a local 2-player territory-capture game built with vanilla HTML5 Canvas + ES6 classes. No frameworks, no bundlers, no package.json at root. Open `index.html` directly in a browser to run local multiplayer.

Online multiplayer (room-based, central server) is **partially implemented** — client-side networking (`NetworkManager`, `RemotePlayer`) and the Node.js server skeleton exist, but the full state-sync pipeline is still being wired up.

Recent performance work replaced most per-frame vector drawing with **pre-rendered sprite images** (`/image/*.png`) loaded by `ImageLoader`, plus distance-squared collision math and batch trail trimming. See `plans/optimization-images.md` for the sprite list and generation prompts.

## How to Run

```bash
# Local game — just open index.html in a browser, no build step.

# Online multiplayer server (also serves static game files on port 3000):
cd server
npm install
npm start        # production — visit http://localhost:3000
npm run dev      # development (--watch auto-restart)
# → WebSocket: ws://localhost:3000
# → Health check: http://localhost:3000/health
# → The server doubles as a static file server — no separate npx serve needed.
```

There are no tests, no linting, no build pipeline.

## Script Loading Order (Critical)

`index.html` loads scripts in a specific sequence. **Load order matters** — each file depends on ones loaded before it:

1. `js/Config.js` — game constants (depends on nothing)
2. `js/modes/Flag.js`, `js/modes/Survive.js`, `js/modes/DayNight.js` — mode logic
3. `js/objects/Player.js`, `js/objects/RemotePlayer.js`, `js/objects/Territory.js`, `js/objects/UIManager.js`, `js/objects/ItemManager.js`, `js/objects/Color.js`
4. `msgpack-lite` CDN (external, for binary WebSocket messages)
5. `js/core/InputHandler.js`, `js/core/ImageLoader.js`, `js/core/Renderer.js`, `js/core/NetworkManager.js`, `js/core/GameEngine.js`, `js/core/main.js`

**`ImageLoader.js` must load before `Renderer.js`** — the Renderer reads `window.imageLoader` to pick sprite vs. procedural fallback. If adding a new JS file, insert it at the correct position in this chain. A file cannot reference classes defined in files loaded after it.

## Architecture

### SPA Page Navigation

The game uses a single-page app pattern: all "pages" are `<div>` elements in `index.html` with class `page`. Navigation toggles via `page.classList.add('active')` / `remove('active')`. Page navigation functions (`showMainMenu()`, `showOnlineLobby()`, `startGame()`, etc.) are **global functions** defined in `js/core/main.js` and called via `onclick` attributes in the HTML. There is no router — this is direct DOM manipulation.

### CSS Organization

`css/main.css` imports all modules via `@import`:

- `css/bases/` — base, background, components, buttons, forms
- `css/pages/` — color-picker, rules, games, game-ui, overlays, online-lobby

Each page/overlay has its own CSS file. When adding a new UI page, add its CSS in `css/pages/` and import it in `main.css`.

### Game Loop (GameEngine.gameLoop)

Uses a **fixed timestep** pattern:
- Physics runs at 60 FPS (`PHYSICS_TIMESTEP = 16.67ms`), max 5 frame skips
- Rendering is decoupled — variable frame rate with an interpolation factor
- `fixedUpdate(deltaTime)` is the authoritative tick; `render(interpolation)` is visual only

### State Machine

`gameState` transitions: `menu` → `countdown` (3s) → `playing` → `paused` / `gameOver`

### GameEngine Dual Init Pattern

`GameEngine` has two initialization paths:
- **`init(canvas)`** — local 2-player mode. Creates renderer, item manager, territory canvas, static/dynamic offscreen canvases, and sets `networkMode = 'local'`.
- **`initOnline(canvas, networkManager, localPlayerId)`** — online mode. Sets `networkMode = 'online'`, `isServerAuthoritative = true`. The game loop switches to `updateOnline()` which sends inputs to the server and interpolates remote players instead of running local collision detection.

### InputHandler Dual Mode

- **Local mode**: Processes two sets of key bindings (`Config.KEY_BINDINGS.player1` = WASD, `player2` = Arrow keys) simultaneously. Both players' inputs update both Player objects directly.
- **Online mode**: `setLocalBindings(bindings)` locks to a single binding set (typically player1/WASD). The local player's input is sent to the server; remote players are driven by server state, not keyboard.

### Canvas Compositing (3 Layers)

The Renderer uses three offscreen canvases composited onto the main canvas each frame, **bottom → top**:

1. **Territory canvas** — filled enclosed polygons; updated only when `territoryChanged === true`
2. **Static canvas** — spawn points, items, obstacles, barriers, flags; redrawn every 3 frames or when `staticNeedsUpdate === true`
3. **Dynamic canvas** — trails + player tanks + AI enemies; redrawn **every frame** when `dynamicNeedsUpdate === true`

Compositing order (in `GameEngine.render`) is: `territory → static → dynamic`, so **trails and tanks draw ON TOP of spawn points and flags**. This layering avoids expensive full redraws. The `item/obstacle/barrier` rendering uses per-state cached offscreen canvases with a 60-second expiry.

### Sprite System (ImageLoader + /image)

All static, repeatedly-drawn shapes are pre-rendered PNG sprites in `/image/` and drawn via `ctx.drawImage` instead of vector paths/gradients:

- **`js/core/ImageLoader.js`** — global `window.imageLoader` singleton; preloads every sprite into `HTMLImageElement`s. `isLoaded(name)` / `get(name)` return falsy for missing/failed images, so callers fall back to procedural drawing.
- **`/image/*.png`** — 12 sprites (see `plans/optimization-images.md` for the mapping + prompts): obstacle, initial_barrier, item_speed/length/shield, tank_template, spawn_point_template, moon, flag, enemy_red, enemy_relax, shield_ring.
- **Grayscale tinting**: `tank_template` and `spawn_point_template` are pure-grayscale templates tinted at runtime via `Renderer.getTintedSprite(name, color, region)` — it draws the sprite, then `multiply`-composites `color` with `globalAlpha = 0.6` (so white areas aren't fully saturated). Results are cached in `this.tintCache` (keyed by name+color+region, capped at 50).
- **Region-limited tinting**: `getTintedSprite` accepts a `region` param:
  - `{shape:'circle', radius}` — clips tinting to a circle (spawn points, flag emblem), keeping corners untinted.
  - `{shape:'rect', x, y, w, h}` — clips tinting to a rectangle. The tank uses `{x:0,y:0,w:43,h:32}` so the **gun barrel (sprite x ≥ 43) stays grayscale, untinted**.
- **Pre-colored sprites**: `enemy_red` (active AI) and `enemy_relax` (5s post-hit relax state, light blue) are already colored — drawn directly, no multiply. `Survive.AIEnemy.render` picks the sprite by `relaxMode`, falling back to procedural for relax only if the image is missing.

### Global Singleton Pattern

`window.gameEngine` is the global GameEngine instance. Many classes (Player, ItemManager, UIManager) reference `window.gameEngine` directly to check game state (`gameState`, `gameMode`, `paused`). When refactoring, this coupling must be handled — you can't simply pass gameEngine as a constructor parameter to all classes without updating every reference.

### Victory Condition Chain

Winners are determined by this priority order:
1. **Lives** — player with more remaining lives wins
2. **Flags** (capture mode only) — more captured flags wins
3. **Territory area** — larger enclosed area percentage wins
4. **Draw** — all three equal → tie

### Collision System (Mode-Dependent)

Collision rules change per `gameMode` inside `GameEngine.checkCollisions()`:

| Mode | Head-on collision | Trail collision |
|------|-------------------|-----------------|
| `explore` / `capture` | Both die | Collider dies |
| `fight` / `infinite` / `daynight` | Both die | Trail owner dies |
| `survival` | N/A (AI-managed separately) | N/A |

Additionally, entering enemy territory kills the entering player in all modes (unless shielded). Collision checks use **squared distances** (no `Math.sqrt`) for speed; `GameEngine` deduplicates the player-pair scan so each pair is checked once.

### Player Class Design

- `Player.fixedUpdate(deltaTime)` controls movement, trail sampling (at `standarddistance = 2px` intervals), and home-countdown logic for capture mode. Trails are trimmed with a single `splice` batch (not per-point `shift()`) to avoid O(n²).
- `Player.setDirection(dx, dy)` **normalizes the direction vector** — diagonal inputs like `{1,1}` become `{0.707, 0.707}`, so all 8 directions move at identical speed. The server (`ServerEngine.setDirection`) applies the same normalization, and the anti-cheat speed check is unaffected (normalized magnitude = 1).
- `Player.die(reason)` decrements lives; in fight/survival/daynight/infinite modes, calls `diereset()` instead of killing permanently
- `diereset()` reinitializes position/trail but preserves score; in infinite mode, applies increasing lockTime
- `Player.canEnterEnemyTerritory()` returns `true` only when the player has an active shield

### Territory Detection

`Territory.detectEnclosure(player)` — when the player's head comes within `homeRadius` (20px) of their start point:
1. Combines trail + current position + start position into a closed polygon
2. Uses the shoelace formula to determine winding order
3. Standardizes to counter-clockwise, then stores in `this.areas` Map
4. `calculateScore()` rasterizes territories onto an offscreen canvas and counts filled pixels (sampled every 2px)

### Mode Managers (Client)

- **FlagManager** (`modes/Flag.js`): Places 5 flags, tracks ownership by last player to enclose each flag's position. Uses the `flag.png` sprite tinted by ownership color, **clipped to a circle** so the tint doesn't bleed into corners
- **SurvivalManager** (`modes/Survive.js`): Spawns AI enemies with wander/chase/avoid/relax states. AI detection radius = 150px. After hitting a player, enters 5s relax mode (light blue `enemy_relax` sprite, no chase)
- **DayNightManager** (`modes/DayNight.js`): 20s cycle (5s fade dark → 10s dark → 5s fade light). Visual overlay only; doesn't affect gameplay logic

### Color System (Color.js)

`ColorManager` provides:
- 7 preset colors per player (red, blue, green, yellow, orange, cyan, olive)
- Custom color picker with hex input
- HSL slider adjustment (hue, saturation, lightness)
- Color harmony indicator that evaluates distinguishability between player colors
- 6-entry color history per player

### Item System

`ItemManager` spawns power-ups (speed/length/shield) every 2s and obstacles every 5s. Items timeout after 10s. Max 30 obstacles. Initial barriers spawn at game start and disappear after 12s. Power-up weights: speed 40, length 30, shield 30.

## Online Multiplayer (Partial Implementation)

The design is documented in `plans/online-multiplayer-design.md`. Implementation has begun:

### Server (`server/`)
- **Entry**: `src/index.js` — creates HTTP + WebSocket server on port 3000. **Serves static game files** from the project root (path-traversal-safe), so the server alone hosts the full game. Also exposes `/health` and `/api/rooms`.
- **ConnectionManager** (`src/network/ConnectionManager.js`): WebSocket lifecycle, heartbeat (10s ping, 5s pong timeout, 30s disconnect), per-player message routing
- **RoomManager** (`src/rooms/RoomManager.js`): Room CRUD, room code generation (6-char), public room listing
- **Room** (`src/rooms/Room.js`): Player slots, ready state, game start/end lifecycle
- **ServerEngine** (`src/engine/ServerEngine.js`): Authoritative server-side game simulation (no rendering). 20 Hz tick rate. Direction normalization (matching client) lives in `ServerEngine.setDirection`.
- **Server mode managers**: `ServerSurvivalManager.js`, `ServerDayNightManager.js`, `ServerFlagManager.js` — server-side mirrors of the client mode managers, running authoritative game logic without DOM/Canvas dependencies
- **MessageTypes** (`src/network/MessageTypes.js`): Protocol constants (C2S_*/S2C_*), game events, error codes, plus `buildMessage()` (JSON) and `buildBinaryMessage()` (MessagePack) helpers
- **config.js**: Server tick rate 20 Hz, anti-cheat thresholds (speed tolerance 10%, max 3 violations, 120 max input/s), reconnect window 60s
- Dependencies: `ws`, `uuid`, `msgpack-lite`

### Client Networking
- **NetworkManager** (`js/core/NetworkManager.js`): WebSocket client, message send/receive (JSON + MessagePack binary), ping measurement, exponential backoff reconnect (max 5 attempts), input sequencing for server reconciliation, event callback system
- **RemotePlayer** (`js/objects/RemotePlayer.js`): Interpolation buffer (adaptive 30-200ms delay), jitter measurement from state arrival intervals, smooth delay transitions, linear interpolation between buffered positions
- **Online Lobby** (`css/pages/online-lobby.css`, plus HTML in `index.html`): Room creation/joining UI, player slots, ready toggling, ping display

### Architecture (from design doc)
- **Server-authoritative**: All collision, territory, and scoring logic runs on the server
- **Client-side prediction**: Local player moves immediately; server corrections applied when error > 5px
- **Remote player interpolation**: 100ms default interp delay, adaptive to network jitter
- **State broadcast**: 20 Hz from server, full + delta snapshots
- **Rollback netcode** (planned): Server keeps 120-frame history for late-input reconciliation

### Online Game Lifecycle

1. `showOnlineLobby()` → connects WebSocket to server
2. Create/join room → `showPage('roomWaiting')` → ready up
3. Host clicks start → server sends `S2C_GAME_START` with countdown + player list
4. `startOnlineGameSession(data)` → `gameEngine.initOnline(canvas, networkManager, playerId)` → `setupOnlinePlayers(data)` → `gameLoop()` with `updateOnline()` path
5. Game loop sends inputs at 60 Hz, receives server state at 20 Hz, interpolates remote players
6. On game end → `showOnlineGameOver(result)` → player can return to room or leave
7. On main menu while online → `networkManager.forfeit()` + `cleanupOnlineGame()`

Pause is **disabled** in online mode. `restartGame()` in online mode sends forfeit and returns to room waiting.

## Key Configuration

All tunable constants in `Config.js` static properties. Important categories:
- `DIFFICULTY_LEVELS`: speed, maxTime, trailLength per difficulty tier (slow/medium/fast/ultra)
- `PLAYER_DEFAULTS`: standarddistance (trail sampling density), pushdistance, AI params, per-mode lives, locktime, homeTimeLimit
- `CANVAS`: 1200×600, spawn points at (15%, 50%) and (85%, 50%)
- `PHYSICS_SETTINGS`: 60 FPS, max 5 frame skips
- `DAYNIGHT`: cycleTime 20000ms, fadeTime 5000ms, darkTime 10000ms
- `ITEM_MANAGER_SETTINGS`: spawn intervals, durations, max counts, power-up weights/colors/durations
- `KEY_BINDINGS`: player1 (WASD), player2 (Arrow keys)

**Note**: Config values are duplicated in `server/src/engine/ServerEngine.js` (`CONFIG` constant) — server cannot import client-side Config.js. Keep both in sync when changing game parameters.

## Plans Directory

Design documents tracking the online multiplayer implementation and performance work:
- `plans/online-multiplayer-design.md` — comprehensive architecture, protocol, state sync, anti-cheat, deployment, 3-phase roadmap
- `plans/01-client-prediction-reconciliation.md` — client prediction and server reconciliation details
- `plans/02-adaptive-interpolation.md` — adaptive interpolation for remote players
- `plans/03-multi-mode-online-support.md` — multi-mode support in online play
- `plans/05-anti-cheat-system.md` — anti-cheat mechanisms
- `plans/06-binary-protocol-delta-compression.md` — binary protocol and delta compression
- `plans/optimization-images.md` — sprite list + generation prompts for the `/image` sprites, and which shapes stay procedural

---
> Source: [yao1145/quandiyundong](https://github.com/yao1145/quandiyundong) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
