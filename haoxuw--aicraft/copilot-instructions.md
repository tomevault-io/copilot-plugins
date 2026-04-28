## aicraft

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repo layout

```
src/platform/      C++ engine (logic/, net/, server/, agent/, client/, debug/, shaders/, fonts/, docs/)
src/artifacts/     Python game content — hot-loadable, no rebuild needed
                   behaviors/ living/ items/ blocks/ models/ worlds/ structures/ effects/ resources/ annotations/
src/python/        Python helpers (behavior_base, conditions_lib, local_world, pathfind, action_proxy, …)
src/config/        Runtime config (controls.yaml — copied into build dir)
src/resources/     Shipped assets (textures, sounds, models)
src/tests/         test_e2e.cpp, test_pathfinding.cpp, behavior_scenario_validation/
src/model_editor/  Authoring tooling (modelcrafter, bbmodel_export/import, capture_samples.sh)
docs/civcraft_legacy/  Game design docs (overview, actions, physics, inventory, …)
```

**Read `docs/civcraft_legacy/00_OVERVIEW.md` before making ANY gameplay changes.**
For engine/protocol/behavior-API details see `src/platform/docs/` (14_ARCHITECTURE_DIAGRAM.md, 21_BEHAVIOR_API.md, 04_SERVER.md, 06_NETWORKING.md).

## Mandatory Design Rules

These rules are NON-NEGOTIABLE. Every code change must respect them.

### Rule 0: The Server Accepts Exactly Four Action Types ← HIGHEST PRIORITY

**This is the most important architectural invariant. It overrides any other
design consideration.**

The server validates exactly four `ActionProposal` types — nothing more, nothing
less. Every action any entity can ever take must compile down to one of these:

```
TYPE_MOVE     (0) — set entity velocity/direction (no tunnelling)
TYPE_RELOCATE (1) — move an Object between containers (inventory ↔ ground ↔ entity);
                    value is conserved, nothing created from nothing
TYPE_CONVERT  (2) — transform an Object from one type to another (value must not
                    increase); consuming → convert to nothing; casting effect →
                    convert item to hp/effect; empty to_item = destroy
TYPE_INTERACT (3) — toggle interactive block state (door, button, TNT fuse)
```

**Consequences:**
- `Follow`, `Flee`, `Wander` are **NOT action types** — they are Python helpers
  that compute a target position and return `MoveTo` (TYPE_MOVE).
- All AI decision logic (follow, flee, wander, pathfind) lives in Python behaviors,
  never in the C++ bridge or server.
- The C++ bridge (`platform/server/python_bridge.cpp`) must never resolve or interpret
  high-level action names — it only translates the 4 primitives into `ActionProposal`.

See `docs/civcraft_legacy/00_OVERVIEW.md` § Action Types and `docs/civcraft_legacy/03_ACTIONS.md`.

### Rule 1: Python Is the Game, C++ Is the Engine

**All game content and gameplay rules are defined in Python. C++ only provides
the engine (physics, networking, rendering).**

- Creature definitions, behaviors, items, blocks, effects, models → `src/artifacts/`
- **NEVER hardcode gameplay constants in C++.** Every magic number (distance,
  speed, timer, radius) MUST be configurable from Python.
- **Creature AI behaviors are game logic** — they live in Python (`src/artifacts/behaviors/`).
- **Player click-to-move** uses server-side greedy steering (`platform/server/pathfind.h`)
  for simplicity and reliability. Creature pathfinding still runs in Python
  (`src/python/pathfind.py`) via AgentClient. Navigation tuning lives in `ServerTuning`.

### Rule 2: The Player Is Not Special

**Entity = Living + Item.** That's it.

- **Living** — moves, has HP, has inventory. Players, NPCs, animals are all Living.
  The player is just a Living entity with `playable=true`. Any Living can be
  played by hijacking its agent client. NPCs are Living with a BehaviorId.
- **Item** — on the ground or in an inventory.
- **Blocks** are NOT entities — they live in the chunk grid. Some blocks (chests)
  have inventories managed separately, keyed by block position.

### Rule 3: Server-Authoritative World Ownership

**The server is the sole owner and modifier of world state.**

- Entities submit `ActionProposal` (intent). Server validates and executes.
- Client NEVER writes to `chunk.set()`.
- **Client-side prediction**: the GUI client runs `moveAndCollide()` locally
  for the player entity (same physics as server) and reports `clientPos` in
  Move actions. The server accepts `clientPos` if within tolerance (8 blocks),
  otherwise snaps back.
- Singleplayer uses the same TCP code paths as multiplayer — no in-process shortcut.

### Rule 4: All Intelligence Runs on Agent Clients

**The server has ZERO intelligence.** All AI runs on agent clients.

- The AgentClient now lives **inside the player-client process** (one per client,
  not one per entity — see `platform/client/game_vk.cpp` where `m_agentClient` is
  constructed). It receives world state over TCP, runs Python `decide()` for every
  NPC owned by this client, and submits `ActionProposal`s back to the server.
- Python behavior code NEVER runs on the server.
- **Player click-to-move navigation** is handled server-side (greedy local
  steering in `platform/server/pathfind.h`). The GUI client sends `C_SET_GOAL` /
  `C_SET_GOAL_GROUP`; the server sets entity velocities each tick. Player
  entities do NOT need Python AI.
- `TestServer` (`platform/server/test_server.h`) exists only for headless unit tests.

### Rule 5: Server Has No Display Logic

**The server never decides what to show on any client's screen.**

Server responsibilities: validate actions, update world state, broadcast via TCP
(`onBlockChange`, `onEntityRemove`, `onInventoryChange`). Nothing else.

**NEVER add to `ServerCallbacks` for** visual effects, floating text, HUD notifications,
or per-player display decisions. Each client observes the TCP state stream and decides
its own display:
- Block break text → client detects `S_BLOCK` bid→AIR, looks up block name
- Damage text → client compares HP snapshots from successive `S_ENTITY` messages
- Sounds, particles → client-side, triggered by client-observable events

### Rule 6: Unified Physics, One Source of Truth

**See `docs/civcraft_legacy/10_CLIENT_SERVER_PHYSICS.md`.**

- **One tick loop** on the client for ALL entities (player + NPCs). No
  `tickPlayer()` vs `tickNPC()` — there is `tickEntities()`.
- **No dual state.** Entity position lives in `entity.position`. No parallel
  `m_player.pos`. The player is just an entity whose input comes from the keyboard.
- **LocalWorld** (`platform/client/local_world.h`) is the client's single chunk
  store (`ChunkSource`). Shared by the player tick, agent ticks, chunk mesher,
  and raycasting.
- **`moveAndCollide()`** in `platform/logic/physics.h` is the ONE physics
  implementation. Client and server both call it — the only difference is what
  backs `BlockSolidFn`: `LocalWorld` (client, incomplete) vs `World` (server, complete).
- **Reconciliation is uniform.** The player and owned NPCs are reconciled against
  server broadcasts with the same logic. No player special-casing.

## Architecture

Two process types — **always TCP**, same architecture for singleplayer and multiplayer:

| Process | Binary | What it does |
|---------|--------|-------------|
| World Server  | `civcraft-server`  | Headless, owns world, NO OpenGL/Vulkan, NO Python AI |
| Player Client | `civcraft-ui-vk`   | GUI (Vulkan), renders world, hosts AgentClient → runs Python AI for owned NPCs |

Singleplayer: `civcraft-ui-vk` spawns `civcraft-server` as a child process then connects
via localhost TCP. There is no in-process shortcut.

## Build & Run

The root `Makefile` is the one source of truth. **Parallelism is capped at half the
core count** (`PAR := nproc/2`) so a build won't pin the machine or OOM. Override:
`make build PAR=8`. All `cmake --build` invocations should use `-j$(PAR)`.

Two build trees:
- `build/` — `Debug`. Used by `make build`, `make client`, `make server`, `make test_e2e`, visual QA scenarios.
- `build-perf/` — `Debug + CIVCRAFT_PERF=ON` (perf instrumentation compiled in). Used by `make game` (singleplayer iteration) and `make profiler`.

```bash
make game                         # singleplayer via build-perf/  (spawns server, --skip-menu)
make game GAME_PORT=7890          # singleplayer on a fixed port
make server PORT=7777             # dedicated server (interactive world select)
make client HOST=X PORT=N         # GUI client → menu or pre-filled join
make stop                         # kill all civcraft-* processes (exact-name match)
make test_e2e                     # headless gameplay + pathfinding tests
make pathfinding_test             # just the pathfinding regression suite
make proxy                        # HTTP→TCP action proxy w/ Swagger UI on :8088

# Manual cmake (matches `make build`):
cmake -B build -DCMAKE_BUILD_TYPE=Debug
cmake --build build -j$(PAR)

# Direct invocation:
./build/civcraft-ui-vk --skip-menu                         # singleplayer
./build/civcraft-server --port 7777                        # dedicated server
./build/civcraft-ui-vk --host 127.0.0.1 --port 7777        # network client
```

## Iterative Development

**Don't `pkill -f civcraft`** — it matches the shell's own command line (which
contains the word "civcraft") and SIGTERMs your own bash, silently skipping the
rest of the chained command. Use exact-name matching:
`pgrep -x civcraft-ui-vk | xargs kill` (and the same for `civcraft-server`).

**Screenshots** — written by the Vulkan client on request:
- `touch /tmp/civcraft_screenshot_request` → immediate screenshot → `/tmp/civcraft_screenshot_N.ppm`
- **F2** in-game: manual screenshot

**Visual QA without an interactive window:**
```bash
# Items: FPS/TPS/RPG/RTS + on-ground shots
make item_views ITEM=base:sword
# or: ./build/civcraft-ui-vk --skip-menu --debug-scenario item_views --debug-item base:sword

# Characters: 6-angle orbit (front/three_q/side/back/top/rts)
make character_views CHARACTER=base:pig
# or: ./build/civcraft-ui-vk --skip-menu --debug-scenario character_views --debug-character base:pig

# Full sweep across all characters × clips:
make animation_sweep    # writes /tmp/anim_review/<char>/<clip>.png
```

Both write `/tmp/debug_N_<suffix>.ppm` and auto-exit. See `.claude/skills/refine-model-and-animation/SKILL.md` for the full iteration loop.

**Behavioral QA without a window (headless log mode):**

Use this instead of screenshots when you need to verify *what creatures are
deciding and doing*, not what the world looks like. The log is a WoW-style
combat/event stream derived entirely from the TCP state stream (Rule 5
compliant — no server-side logging).

```bash
./build/civcraft-ui-vk --skip-menu --log-only      # singleplayer, hidden window
./build/civcraft-ui-vk --log-only --host H --port P # attach to remote server
# Streams events to stdout AND /tmp/civcraft_game.log (truncated on start;
# prior session preserved as /tmp/civcraft_game.log.prev)
```

In GUI mode the same log is also written to `/tmp/civcraft_game.log` and viewable
from **Main Menu → Game Log** and the pause menu. One source of truth; Claude
reads the file.

**Log format** — `[HH:MM:SS] [CATEGORY] <actor> <event>`:
| Category  | Source                                   | Example |
|-----------|------------------------------------------|---------|
| `DECIDE`  | `goalText` deltas in `S_ENTITY`          | `woodcutter#7 → MoveTo tree@(45,60,30)` |
| `MOVE`    | position deltas in `S_ENTITY` (throttled)| `chicken#5 @ (32,64,25)` |
| `ACTION`  | `S_BLOCK` break/place                    | `woodcutter#7 broke base:log@(45,60,30)` |
| `COMBAT`  | HP deltas across `S_ENTITY` snapshots    | `wolf#9 → sheep#3 for -5hp (12→7)` |
| `DEATH`   | `S_REMOVE` with last HP=0                | `sheep#3` |
| `INV`     | `S_INVENTORY` diffs                      | `player#1 picked up base:log ×1` |

**Verification recipe for behavior/AI/pathfinding work** (preferred over screenshots):
```bash
make build && pgrep -x civcraft-ui-vk | xargs -r kill; sleep 0.5 && \
  ./build/civcraft-ui-vk --skip-menu --log-only &
sleep 10 && pgrep -x civcraft-ui-vk | xargs -r kill
# then Read /tmp/civcraft_game.log — grep for DECIDE/ACTION/COMBAT
```

Use screenshots only when the bug is visual (rendering, animation, UI).

**In-game shortcuts:**

| Key | Action |
|-----|--------|
| F2  | Screenshot |
| F3  | Debug overlay (FPS, XYZ, chunk, entity count) |
| F12 | Toggle admin/survival mode |
| V   | Cycle camera: FPS → TPS → RPG → RTS |
| Tab | Inventory |
| Esc | Pause / back to menu |

## Source structure (detail)

```
src/platform/
  logic/     Pure data types (entity, chunk, action, inventory, physics, constants). No deps. Linked by all.
  net/       TCP protocol (net_protocol.h, server_interface.h, net_socket.h). Depends on logic/.
  server/    GameServer (server.h/cpp), ClientManager, EntityManager, python_bridge.cpp,
             pathfind.h (player greedy steer), world gen, weather, world_save. No GL, no Python except via pybind.
  agent/     AgentClient runtime (embedded in client process): agent_client.h, behavior_executor.h,
             decide_worker.h, pathfind (Python-backed), decision_queue.h. Links Python.
  client/    Vulkan client: game_vk.cpp (main loop), game_vk_playing.cpp (player tick/input/combat),
             game_vk_render.cpp (rendering), local_world.h (client ChunkSource), chunk_mesher.cpp,
             model_loader, network_server.h, process_manager.h (spawns server for SP), rhi/, ui/.
             NO Python imports directly — AI runs via the embedded AgentClient.
  debug/     Diagnostics — crash_log.h, move_stuck_log.h, entity_log.h
  shaders/vk/  GLSL for the Vulkan pipeline (chunk_terrain.frag, sky.frag, …)
  fonts/  docs/  engine-level assets + architecture docs

src/tests/
  test_e2e.cpp          Headless gameplay tests → civcraft-test (uses TestServer)
  test_pathfinding.cpp  Pathfinding regression  → civcraft-test-pathfinding
  behavior_scenario_validation/  Scenario-driven behavior checks

src/artifacts/          Python game content — hot-loadable, no rebuild needed
  behaviors/  Creature AI: decide() functions
  living/     EntityDef: stats, model, behavior
  items/  blocks/  effects/  actions/  resources/  structures/  models/  annotations/
  worlds/     World templates (flat, village)
```

### Dependency rules
- `platform/logic/` → nothing (pure types, linked by all)
- `platform/net/` → `logic/` (protocol types)
- `platform/server/` → `logic/` (no GL, no Python except via pybind)
- `platform/agent/` → `logic/` + `server/behavior.h` + Python
- `platform/client/` → `logic/` + `agent/` (embeds AgentClient); NO direct Python, NO server ownership

## Code Style

- Tabs for indentation in C++ and GLSL
- `namespace civcraft { }` wraps all C++ code; `namespace civcraft::net { }` for protocol types
- String IDs: `"base:name"` format (namespace:identifier)
- Header-only for small classes; `.h` + `.cpp` split for larger ones
- When adding a new `BehaviorAction` type: decide if one-shot or continuous.
  One-shot (creates entities, modifies blocks, deals damage) → add to `extractOneShots()`
  in `platform/agent/behavior_executor.h`.
- **Header-only changes don't trigger rebuild**: after editing a header-only
  content file, `touch` its including `.cpp` to force recompilation
  (e.g. `touch src/platform/server/builtin.cpp`).

## Game identity

**This game is called CivCraft.** Players can mod ANYTHING and EVERYTHING. Every
creature, behavior, item, block, world, and effect is defined in Python artifacts
— fully replaceable, extendable, and shareable without touching C++. The C++
engine is a pure platform; all game identity lives in `src/artifacts/`.

Every feature decision must ask: *"Can a modder override this from Python?"*
If not, it needs to move to an artifact.

## Commit guidelines

- Present tense, capital first letter, under 70 chars
- Area prefix: `server:`, `client:`, `agent:`, `logic:`, `net:`, `artifacts:`, `content:`, `repo:`, etc.

---
> Source: [haoxuw/AiCraft](https://github.com/haoxuw/AiCraft) — distributed by [TomeVault](https://tomevault.io/claim/haoxuw).
<!-- tomevault:4.0:copilot_instructions:2026-04-18 -->
