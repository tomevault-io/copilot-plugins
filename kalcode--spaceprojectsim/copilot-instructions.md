## spaceprojectsim

> Context for Claude Code (claude.ai/code) when working in the `simulator-rust/` project.

# CLAUDE.md

Context for Claude Code (claude.ai/code) when working in the `simulator-rust/` project.

## What this is

Rust space-economy simulator + native Bevy client. The simulation engine
(`sim_core`) is a port of an Elixir/Phoenix prototype — moved off BEAM
because its scheduler performed poorly on Windows gaming PCs. The client
(`client_bevy`) embeds the engine directly and renders + inspects it in
one process, no socket boundary.

Ships at ~282 ships + ~95 facilities + 27 pops + 60 bodies + 8 factions
(~485 agents) today, designed to scale to 100k+. Single native binary,
no runtime dependencies. The Godot client referenced in older history
sections is archived under `client/` as a parity reference and is no
longer the canonical front end — every new feature lands in
`client_bevy`.

See [`docs/ARCHITECTURE.md`](./docs/ARCHITECTURE.md) for mermaid
diagrams of the tick loop / AI stack / economic feedback loop /
runtime ownership / state machine, and [`plan.md`](./plan.md) for
the bundled improvement roadmap. `git log --oneline` is the source
of truth for recent activity.

## Status

**Phases 1–20c + Bundles 1–2 + GOAP arc + Bundle-4 tranche (S/H/G/Q/K) complete.**
190 tests passing, strict verify clean, runtime p50 ≈ 10–20 ms /
p99 ≈ 30–50 ms at 485 agents with WORLD_SPEED=4 (125 ms budget).
See [`plan.md`](./plan.md) for the source-of-truth improvement
roadmap.

**Bundle 1 (starter wins):** price memory decay on `best_trade_route`
(freshness half-life 1500 ticks), shortage-scaled delivery contract
quantity, personality drift on delivery success + rest-timeout,
facility + population LOD gating so off-screen agents skip heavy
sub-phases.

**Bundle 2 (AI planner):**
- 2.1 — **Unified utility scorer** replaces the priority waterfall.
  Every goal scored uniformly with tier weights (bio 100 /
  housekeeping 40 / strategic 3 / explore 6 / idle floor); the
  waterfall emerges from scores rather than hardcoded bypass.
  `FUEL_URGENCY_ONSET = 0.45` softened from the old 0.25 cliff.
- 2.2 — **Mid-action replan.** `ShipAI` gains `current_goal_score`
  + `last_plan_tick`; every `REPLAN_CHECK_INTERVAL_TICKS=20` a
  running plan may be interrupted if an alternative scores
  `× REPLAN_SCORE_RATIO=1.30`+ better, gated by
  `REPLAN_MIN_COMMIT_TICKS=40` hysteresis. Biological + retrofit
  + combined-goal plans are replan-protected.

**GOAP arc (replaces every hand-authored BT builder):**
- MVP — forward planner `sim_core/src/ai/planner.rs` with abstract
  `WorldState` (symbolic `Loc` + docked/fuel_bucket/cargo_loaded/
  delivered/sold/retrofitted/rested flags), `Operator` catalog,
  bounded-depth search. Wired to `build_fulfill_delivery_bt`.
- Expand — trade-run catalog (SellAll terminal) + first combined
  goal `fulfill_delivery_with_retrofit` that folds a shipyard
  detour into a delivery BT. Plain BT builders stay as fallbacks.
- Embrace — real fuel state threaded through via
  `fuel_fraction_to_bucket(ship.fuel.fraction())`. Passenger /
  courier catalog (`DeliverPassengers` operator, no cargo
  mechanics). Standalone retrofit catalog. Second combined goal
  `trade_run_with_retrofit`.
- Complete coverage — crew_mission (`Rest` operator + `rested`
  state, ordering enforced by the `delivered && rested` goal
  predicate) and rest_crew catalogs ported. All 7 goal types
  now planner-backed.
- Cost-weighted search — BFS swapped for Dijkstra. `Operator::cost`
  scores travel by fuel-bucket burn (everything else flat 1),
  so combined plans pick cheapest rather than shortest-by-steps.
- Retire fallbacks — all six `build_*_handauthored` siblings
  deleted (-216 lines). Each `build_*` wrapper calls its
  `try_*_planned` counterpart; on the (never-observed in prod)
  `None` result, returns `Idle` BT and replans next tick.

**Bundle 4 tranche:**
- S — **Structural-warning events.** `market_crisis` +
  `market_recovered` (threshold 200 shortage ticks with 50-tick
  hysteresis), `facility_starving` + `facility_recovered`
  (condition <0.3 / >0.5). Emitted from a `scan_crisis_events`
  pass after `tick_markets`, hysteresis-flagged on
  `MarketHistory.crisis_notified` + `FacilityOperations.starving_notified`.
- H — **Facility recipe profitability gate.** `recipes::execute`
  takes optional `prices` and skips batches where
  `output_value < input_value × MIN_RECIPE_MARGIN_RATIO` (0.5,
  so ≥50% loss → skip; normal fluctuation runs through).
- G — **Docking-fee licensing burn.** `commit_dock` burns
  `DOCK_FEE_BURN_PCT=0.30` of every dock fee to the void; 70%
  goes to the facility as before. Primary lever against long-run
  credit inflation.
- Q — **WorldTuning scaffold.** All ten tonight-commit tuning
  constants collected into `sim_core/src/simulation/tuning.rs`.
  Loaded at boot from `priv/data/tuning.json` (missing file →
  defaults; partial JSON → `serde(default)` fills the gaps).
  Inspectable via `GET /api/admin/tuning`. Call sites read
  `WorldTuning::current()`.
- K — **Population wealth as a real variable.**
  `PopEconomy.total_wealth` now responds to production + consumption
  at local market prices (rate 10% — enough to observe drift,
  small enough not to destabilise). Smoke observed producer pops
  (Veritas) gaining + consumer pops losing as expected.
- M — **Throttled snapshot cache rebuild.** Ships + construction
  sites + body/facility positions refresh every tick; full
  market/facility/population/body/contract snapshots every 4 ticks;
  faction snapshots every 16. Debug p50 dropped from ~19ms to
  ~18ms at 485 agents; the gap widens as n grows.
- J — **Convoy-bundled delivery contracts.** Co-located facilities
  posting for the same commodity merge into a single larger
  contract (qty summed, capped at 2×MAX). Rich lead pays the
  combined cost; delivery lands in shared market supply so
  everyone benefits. Observed live at frostheim-station +
  jupiter-station (qty=300 contracts where unbundled would cap
  at 200).

**Bundle 4 deferred (with rationale):**
- P — Contract board index. Current `posted_for_ship` is already
  O(1) per contract; bucketed indexes save ~1ms at 485 agents.
  Revisit if perf HUD flags it.
- O — Spatial grid. At 485 agents × 5 observers, LOD recompute
  is ~2.5k calcs per 50-tick interval — not hot.
- N — Rayon on ship phase. hecs's `&mut World` is thread-exclusive;
  a real parallel path needs snapshot-then-plan-then-apply
  refactor. ~1.5× at current scale, ~5× at 10k. Save for when
  it's genuinely needed.

**Phase 18 (subsystems + self-regulation):** ships got a 5-slot fitting
via 18a, refit via 18b, shipyards as module producers via 18c,
agent-driven retrofit via 18d, facility subsystems via 18e, ship size
tiers + cross-system contract gate via 18f, size speed penalty via 18g,
personality-weighted plan_goal via 18i, contract payment age-escalation
via 18j, morale-zero death spiral fix via 18k (multi-facility dock
fallback, Rest timeout, non-docked recovery), dock-queue awareness +
congestion-aware rest via 18L.

**Phase 19 (Elixir-parity + adaptive systems):**
- 19a — **Facility crew complement** (aggregate struct, efficiency
  multiplier on recipe output, food draw from reserves)
- 19b — **Contract auto-renewal** (Delivery + Construction + Subsidy;
  Subsidy gets a one-shot "desperate bounty" that slams min_rep to 0)
- 19c — **Demand analyzer + construction sites** (faction posts a
  Construction contract for chronic shortages; a `ConstructionSite`
  entity accretes delivered cargo 0→1.0 then materialises into a real
  facility at the same position — persisted across restarts via migration 011)
- 19d — **Population migration** (wired the dormant `migration_pressure`
  field into actual pop movement)
- 19e — **Reputation scaling** (`(5 + payment/2000).clamp(1,15)` on
  delivery; breach penalty `-(10 + payment/1000)` clamped `[-25,-1]`)
- 19f — **Contract-fill-ratio scoring** + urgent tool self-repair
- 19g — **Retirement ledger** (`retired_ships` table, migration 010)
- 19h — **Exploration + smart idle** (idle_ticks → explore at adjacent
  non-recent system; anti-yo-yo via `ShipMemory.recent_systems`)
- **Treasury rebalance** — split floors (`FACTION_WORKING_CAPITAL_FLOOR
  = 10k` for bailouts, `FACTION_SUBSIDY_FLOOR = 5k` for subsidies;
  tax rate 0.5% → 2.5%; bailout floor 3k → 500; bailout target 6k →
  2k). Stopped the money-laundering loop that pinned every treasury
  at $2k.
- **Adaptive fleet** — strike-based retirement: `broke_strike` when
  credits < 500 + cargo empty + no contract; `hoard_strike` when
  credits > 200k + idle_ticks > 500. 3 strikes → decommission +
  `PendingRespawn` 300-600 base ticks later with randomised
  class/faction/location/crew.
- **Admin delete cascades** — `drop_ship_holdings` (Accepted → Posted,
  PickedUp → Expired); facility delete kicks docked ships, cancels
  contracts, clears paired-gate back-references.
- **Dock queue (FIFO)** — `FacilityStorage.dock_queue: Vec<Entity>`;
  queue head gets admit priority, broke heads self-evict, zombie
  entries pruned on both apply-time and a 10×mult-tick global sweep.
  Waiting ships pin to primary facility position so they orbit with
  their station.
- **Capacity rebalance** — world-JSON docking_capacity bumped 545 →
  1595 slots fleet-wide (Terra Prime 50, Haven 45, Anvil 40, etc.;
  min 10 everywhere).
- **world_seed** — persistent u64 minted on first boot (through
  `simulation_state.world_seed`), exposed on `/api/health`,
  `/api/report`, and the `server_hello` WS frame. Both clients seed
  their nebula backdrop off it so landmark layout stays put across
  restarts.

**Phase 20 (observability + ergonomics):**
- 20a — **Command-trace ring** (`AppState.command_trace:
  VecDeque<CommandTraceEntry>` cap 1000; `GET
  /api/debug/agent/:id/trace?limit=N` for sim_server consumers;
  `client_bevy`'s ship-details panel reads the same ring in-process
  and renders a "Recent commands" subsection.)
- 20b — **Partial ship.rs split** (`ship/` directory: `mod.rs` +
  `snapshot.rs` + `actions.rs`; 4167 → 3420 line god module. Planner
  + BT builders still in mod.rs, intentionally deferred as lower ROI
  per in-session review)
- 20c — **Route-performance memory** (`ShipMemory.route_performance:
  HashMap<(from, to, commodity), RouteStats>`; Sell arm records
  realised profit per unit; `best_trade_route` penalises routes where
  actual avg profit < 50% of projected after 3+ attempts)

Earlier (Phases 1–17): spatial LOD for ships (observer-driven tick
gating), inter-system jump gates, full Elixir-parity behaviour:
populations post food + passenger contracts under stress, ships fulfill
them (docking with a fee, multi-tick unloading + refuel, shore-leave
for morale), facilities run recipes, degrade + self-repair, level up,
abandon when chronically broke. Factions tax + subsidise. Crew have
current_task + skills + hunger + morale. Bodies regen resources + biome
multipliers. State persists across restarts. See the **Systems wired**
table below.

## Architecture

Two binaries share `sim_core`:

- **`client_bevy`** (canonical) — embeds the engine. Bevy ECS + egui own
  the main loop; `sim_core::AgentStore` is wrapped as a Bevy `Resource`
  that systems read directly. No socket boundary, no JSON marshalling.
- **`sim_server`** (headless / CI) — wraps the engine in axum + sqlx +
  tokio for WebSocket / HTTP scenarios. Kept for smoke scripts,
  persistence-roundtrip tests, and as a `[lib]` exporting the apply
  helpers `client_bevy` re-uses.

```
client_bevy (bevy + egui + sim_core + sim_db)
  ├── main.rs: AppState { MainMenu, InGame }; Bevy plugins wire-up
  ├── sim/: Sim Resource (= AgentStore + tick + tick_mult + speed),
  │         tick_sim on FixedUpdate (125 ms × tick_mult sub-iters),
  │         load_world OnEnter(InGame), teardown_world OnExit(InGame),
  │         in-process admin processors (spawn / delete / save-as /
  │         supply delta)
  ├── render/: hecs → Bevy entity mirror + per-frame sync,
  │            authored ship GLBs, station GLBs, sun + planet shaders,
  │            nebula backdrop, starfield, hanabi particle trails,
  │            tick interpolation via Time::<Fixed>::overstep_fraction
  ├── ui/: egui inspector panels (perf / ships / markets / factions /
  │        pops / facilities / contracts / construction / retirements /
  │        events / details / minimap / labels / pause / options /
  │        save-list / new-universe), Esc panel-stack, P shortcut
  ├── camera/: orbit camera + WASD pan, follow lock, focus + reset
  ├── audio/: music loop + procedural-baked SFX + pentatonic click chain
  └── icons/: cached egui::Image handles for every panel toggle

sim_server (tokio + axum + sqlx + sim_core + sim_db + sim_protocol)
  ├── engine: tick loop driver + command apply + LOD recompute +
  │           admin spawn / refit dispatch (apply helpers `pub` —
  │           shared with client_bevy via `[lib]`)
  ├── ws/handler: per-client select loop (sim_protocol JSON)
  ├── http/api: read-only inspection + perf + observers + report
  ├── http/admin: ship/facility/market CRUD, save-as
  ├── snapshot: DashMap lock-free read cache for HTTP/WS readers
  └── (no Bevy, no rendering — headless only)

sim_core (pure, no async, no IO)
  ├── agents/*: components + tick fns (hecs::World)
  │   ship/ (mod + snapshot + actions), market, facility,
  │   population, celestial_body, faction, construction
  ├── ai/*: BT<A> generic engine, utility scoring, considerations,
  │         personality, GOAP planner
  ├── economy/*: commodities, order_book, price_engine, recipes,
  │              contracts, missions, module_production
  ├── modules/*: ship subsystem catalog (5 slots × 3 tiers)
  ├── facility_modules/*: facility subsystem catalog (2 slots × 3 tiers)
  ├── simulation/*: config (ShipClass, ShipSize), spawner (AgentStore
  │                 + Pending{Materialization, Decommission, Respawn}),
  │                 commands (SimCommand), lod (AgentLod tier table),
  │                 tuning (WorldTuning)
  └── world/*: registry, travel, body

sim_db (sqlx + tokio + sim_core)
  ├── pool, seeder, ECS loader, persistence worker, migrations 001–011

sim_protocol (serde + sim_core)
  └── WS wire types — sim_server's protocol module, unused by client_bevy
```

### Golden rules

1. **sim_core never imports tokio, sqlx, or axum.** Pure logic only.
   Trivially testable in `#[test]` blocks, reasoned about without async.
2. **hecs is the source of truth during a tick.** `AgentStore` wraps
   `hecs::World` + the `ContractBoard`. The tick loop holds exclusive
   mutable access — no locks. After the tick, `SnapshotCache` is updated
   for HTTP/WS readers.
3. **Cross-phase effects go through `SimCommand`.** A phase reads world
   state, writes commands to a buffer, and never mutates *other* entities
   mid-phase. The next phase drains and applies. Deterministic replay,
   no write conflicts.
4. **Tick phase order is fixed** in `engine::run_tick_loop`:
   ```
   drain-control → bodies → ships →
     record-command-trace → apply-ship-commands →
     drain-pending-materializations (async: DB INSERT for
       construction-completion facilities, backfills
       FacilityIdentity.db_id into ECS) →
     drain-pending-decommissions (async: strike-triggered
       retire → admin_remove_ship + queue PendingRespawn) →
     drain-pending-respawns (async: spawn replacement ship
       with randomised class/faction/location/crew) →
   facilities → populations → markets →
     record-command-trace → apply-ship-commands →
     (drains above run again for market-emitted commands) →
   factions → snapshot → broadcast → emit-persist-writes
   ```

## Crate layout

| Crate | Depends on | Purpose |
|---|---|---|
| `sim_core` | `hecs`, `serde`, `rand`, `rayon` | All simulation logic, pure |
| `sim_protocol` | `serde`, `sim_core` | WS wire types (sim_server only; kept for CI) |
| `sim_db` | `sim_core` + `sqlx`, `tokio`, `anyhow` | Persistence: pool, seeder, spawner, persistence worker |
| `sim_server` | `sim_core` + `sim_db` + `sim_protocol` + `tokio`, `axum`, `dashmap`, `flume` | Runtime binary (WS/HTTP, headless CI use) |
| `client_bevy` | `sim_core` + `sim_db` + `bevy`, `bevy_egui` | **Embedded** Bevy client — sim runs in-process |

### Bevy client (embedded)

Godot client hit a ceiling at ~500 ships (script dispatch across the
scene tree saturated the frame). With the "no multiplayer, ever"
constraint, the WS boundary became architectural dead weight. The
Bevy client treats `sim_core` as a library — `Sim` wraps `AgentStore`
as a Bevy Resource; the render layer queries the hecs `World`
directly; zero marshalling between sim and render. Godot is archived
under `client/` as a parity reference.

#### Current state (post-pivot)

- **AppState machine** — `MainMenu` is the title scene (starfield +
  nebula backdrop, sim paused, world not loaded); `InGame` is the
  running simulation with full HUD. Discrete scenes:
  `sim::load_world` runs on `OnEnter(InGame)` (opens DB pool, seeds
  if empty, loads agents into the in-memory `AgentStore`);
  `sim::teardown_world` runs on `OnExit(InGame)` and resets the store
  so a different universe can be loaded next Start. Game-world
  Bevy entities tagged with `GameWorld` get `despawn_recursive`'d
  on exit. `render::spawn_*` systems are explicitly
  `.after(crate::sim::load_world)` so the AgentStore is populated
  before the renderer reads it.
- **Universe naming** — Start opens a modal with a sanitised name
  field (default `sim`, alphanumeric + `_-` only, 48-char cap).
  Collisions with existing `<name>.db` files auto-append `-2`, `-3`…
  so Start always means *fresh world*; resume goes through Load Game.
- **Save list** — Load Game scans the binary's directory for `*.db`,
  pins the active `sim.db` at the top with a "Current session" badge.
  Both Load and Delete are enabled for any save (deleting `sim.db`
  from the menu is safe — no pool is held, next Start re-seeds).
  Sidecar WAL/SHM/journal cleanup runs on delete.
- **Pause menu** — Esc / `P` / top-bar "Menu" button. Resume / Save As
  / Options / Help / Return to Main Menu / Quit.
  `PauseMenuState.was_running` preserves prior running state across
  the pause overlay so closing doesn't auto-resume an already-paused
  sim.
- **Esc panel-stack** — `OpenPanelStack` resource maintains a
  `Vec<PanelKind>` reconciled each frame from every panel's `open`
  flag. Esc pops the top into `CloseTopRequest`; a separate
  `close_top_panel` system flips that panel's flag. Esc with no
  panels open opens the pause menu. `P` jumps straight to the pause
  menu regardless of stack state.
- **Options panel** — audio sliders for music + SFX volume.
  `apply_music_volume` writes the slider value to `AudioSink.set_volume`
  every frame the resource changes, so music-volume drag is heard
  immediately rather than waiting for the loop point.
- **Tick interpolation** — every ship + body carries an `InterpPos
  { prev, curr }` component. `snapshot_tick_positions` runs in
  `FixedUpdate` after `sim::tick_sim`, rolls `curr → prev`, stamps
  the freshly-ticked position into `curr`. `sync_positions` and
  `follow_facilities_to_bodies` (in `Update`) lerp by
  `Time::<Fixed>::overstep_fraction()` — smooth motion at any FPS,
  independent of the 8 tps × tick_mult cadence.
- **Authored visual assets** — five Godot-source GLB hulls
  (Spitfire / Dispatcher / Imperial / Challenger / Executioner)
  mapped per ship class via `ShipModels`. Stations + gates ship
  with their own GLBs in `crates/client_bevy/assets/stations/`.
  Bodies use custom WGSL: `PlanetMaterial` (procedural surface +
  optional cloud + ring children), `CloudMaterial`, `HaloMaterial`
  (sun corona), `NebulaMaterial`, `StarfieldMaterial`. Asteroid +
  belt bodies get displaced ico-sphere meshes with multi-axis
  `Tumble` rotation.
- **Particles (`bevy_hanabi`)** — engine trails (cyan exhaust,
  per-ship sibling entity, gates on docked state via
  `Visibility::Hidden`), explosions on every despawn path
  (admin Destroy, broke / hoard auto-retire, context-menu delete),
  gate-jump flashes on `ship_gate_jumped` events.
- **Audio** — Bevy 0.14 audio. `AudioPlugin` loops a random
  Sci-Fi track at boot, loads pre-baked `.ogg` SFX (generated by
  `tools/gen_sfx.py` from the Godot client's `procedural_audio.gd`),
  and wires three event-driven hooks: pentatonic click chain
  (6 steps with 1.2 s reset window) on every egui mouse-down,
  explosion (small / med / large by ship class) on
  `ship_decommissioned`, gate-jump ping on `ship_gate_jumped`.
- **Versioning** — `[workspace.package].version` in the workspace
  `Cargo.toml` is the single source of truth. The Rust binary reads
  it as `env!("CARGO_PKG_VERSION")` and shows it in main + pause
  menus (with credits "By Kalcode & Claude / Made with ❤️ in
  Illinois"). The `Makefile` greps the same field with `awk` for
  `Info.plist`, butler `--userversion`, and archive names so
  packaged builds can never drift apart.
- **Distribution** — `Makefile` wraps cargo + rustup + lipo +
  codesign + butler:
  - `make build-mac` (arm64, default) / `build-mac-intel` /
    `build-mac-universal` / `build-win` (MSVC, run on Windows host) /
    `build-linux` (best on Linux host).
  - `make package-mac` (arm64-only `.app` + ad-hoc codesign +
    Info.plist) / `package-mac-universal` (fat binary, requires both
    rustup targets) / `package-win` (zip) / `package-linux` (tar.gz).
  - `make dmg` / `zip-mac` for the macOS deliverable.
  - `make publish-mac` / `publish-win` / `publish-linux` push to
    itch.io via butler; `ITCH_USER` + `ITCH_GAME` overridable on
    the CLI.

#### Bundle history

- **Bundle 2a**: pivot from WS-client to embedded sim skeleton.
- **Bundle 2b.1**: extracted `sim_db` so `client_bevy` doesn't need
  `sim_server`'s HTTP stack.
- **Bundle 2b.2**: load real world via `sim_db` at Startup (one-shot
  tokio runtime for sqlx).
- **Bundle 2c**: mirror hecs entities as Bevy entities (placeholder
  meshes: boxes/spheres/cylinders).
- **Bundle 2b.3**: minimum-viable tick — `ship::tick_ships` runs,
  ships move in transit. Emitted `CommandBuffer` is discarded; full
  apply_ship_commands wiring is deferred as **Bundle 2b.4**.
- **Bundle 3a**: orbit camera (mouse drag orbit, WASD pan, scroll
  zoom, right-drag pan).
- **Bundle 3b**: perf overlay (FPS, tick, counts, world_seed).
- **Bundle 3c**: ship list panel (sortable, filterable, paged).
- **Bundle 3d**: faction-tint on ship meshes (per-faction material).
- **Bundle 3e**: contracts panel (6-kind / 6-status filters,
  show-expired, paged).
- **Bundle 3f**: factions panel (ledger with ship/fac counts +
  treasury + subsidies; pairwise relations matrix).
- **Bundle 3g**: markets panel (collapsing headers per location,
  per-commodity price/supply/demand, crisis/short tier sort).
- **Bundle 3h**: populations panel (sort by count/sentiment/food/
  wealth/migration; colour-coded sentiment + food bands).
- **Bundle 3i**: ship details panel (click-select from ship list →
  full inspector reading every component directly from hecs).
- **Bundle 3j**: facility list + details panel (same pattern;
  abandoned facilities strike-through).
- **Bundle 3k**: "Focus camera" button on details panels
  (`CameraFocusRequest` one-shot Resource, consumed by a camera-side
  system that snaps `OrbitCamera.focus`).
- **Bundle 3l**: retirements panel. `sim_db::load_retirements` loads
  up to 200 rows at startup; panel filters + sorts with
  colour-coded reasons (broke/hoard/admin).
- **Bundle 3m**: construction sites panel. Stacked cards sorted by
  progress descending, egui ProgressBar per site, per-card "Focus"
  button that jumps the camera to the site universe position.
- **Bundle 3n**: selection highlight. `highlight_selected` scales
  the currently-selected ship or facility render entity up to 2.2×
  and resets peers each frame. SelectedShip / SelectedFacility
  promoted to pub so render/ can read them.
- **Bundle 3o**: pause + speed controls in the top bar. Pause flips
  `Sim.running`; speed combo (1×/2×/4×/8×) drives `Sim.tick_mult`,
  and `tick_sim` now loops the tick body that many times per
  FixedUpdate. Top bar also shows the live `tick N` counter.
- **Bundle 3p**: transit line gizmos. Per-frame `draw_transit_lines`
  renders a faint Gizmos line from each in-transit ship's
  `departure_position` to its current position — trail-behind-the-
  ship look. Batches into one draw call.
- **Bundle 3q**: starfield backdrop. 2000 emissive star-spheres
  scattered in a shell around origin, seeded off `Sim.world_seed`
  so constellations stay put across restarts (parity with Godot's
  nebula seed). Shares one mesh + one unlit material → one draw
  call.
- **Bundle 3r**: click-on-mesh raycast selection. Left-click picks
  the nearest ship or facility via `Camera::viewport_to_world` +
  ray-sphere test at a screen-space-scaled radius (~10px at
  typical FOV). Skips egui-consumed clicks; empty-clicks leave
  selection unchanged.
- **Bundle 3s**: directional ship shape + rotation. Ship placeholder
  mesh is now an elongated 1×1×3.2 prism (nose at +Z); in-transit
  ships rotate around Y to face their travel direction, derived
  from `departure_position → current position`.
- **Bundle 3t**: admin Grant XP button. Ship details panel has a
  "+100 XP" button that mutates `ShipProgression` directly via
  hecs `world.get::<&mut _>`, bypassing `SimCommand`. Proves the
  embedded-arch admin pattern: no HTTP, direct component write.
  Applied after the read-only query closure drops its borrow.
- **Bundle 3u**: minimap. Bottom-right egui window draws a top-down
  map — bodies as medium dots, ships as tiny dots, yellow crosshair
  at camera focus, bounds auto-fit to body cluster each frame with
  10% margin. Click anywhere on the minimap to recenter the orbit
  camera on that universe position.
- **Bundle 3v**: per-class ship silhouettes. Placeholder assets now
  carry one Cuboid per class — courier slim+long, freighter wider,
  hauler biggest+squat, tanker medium+tall, miner stubby — so role
  reads at a glance.
- **Bundle 3w**: body labels overlay. `Camera::world_to_viewport`
  projects each celestial body's position to screen, egui draws
  its name above the sphere. Toggle via "Labels" in top bar.
  Default ON for orientation; off-screen + behind-camera bodies
  skipped automatically.
- **Bundle 3x**: admin Delete Ship button + render cleanup.
  Despawns the ship, yanks it from docked_ships + dock_queue on
  every facility (mirrors sim_server admin cascade), and drops
  its contract holdings. Paired `cleanup_stale_ships` render
  system despawns orphan Bevy entities whose sim_id isn't in
  `ship_by_id` anymore.
- **Bundle 2b.4**: full tick chain wired into `client_bevy`.
  `sim_server` grew a `[lib]` target whose `src/lib.rs` re-exposes
  the engine / control / http / ws / snapshot / app_state modules
  as `pub`, and the apply helpers (`apply_ship_commands`,
  `apply_request_dock_typed`, `apply_release_dock`, `commit_dock`,
  `apply_gate_jump`, `apply_facility_refuel`,
  `apply_construction_delivery`, `apply_fitting_swap`,
  `try_shipyard_refit`, `commit_shipyard_refit`,
  `primary_dock_at_location*`, `dockable_candidates_ordered`,
  `prune_stale_dock_queue`, `tick_dock_queue_advance`,
  `tick_bodies`, `tick_relations_pass`, `scan_crisis_events`) flip
  from `fn` to `pub fn`. The `simulator` binary loses its local
  `mod X;` declarations and imports from `sim_server::*` — one
  compilation unit feeds both targets.

  `client_bevy` adds `sim_server` as a path dep and `tick_sim` now
  mirrors `run_tick_loop`'s phase order: bodies → ships → apply →
  dock-queue-advance → facilities → populations → markets → apply
  (2nd pass for market TransferCredits) → bodies_regen →
  scan_crisis_events → factions → relations. Economy is LIVE:
  facility recipes run, markets price, pops consume + migrate,
  factions tax + subsidize, treaties transition. Skipped vs. the
  full loop: the three async drains (materializations /
  decommissions / respawns), persistence emit, WS broadcast.

  Trade-off: `client_bevy` pulls axum / tower / flume / dashmap
  transitively. They compile but aren't exercised. Preferred over
  the cross-cutting risk of moving ~1200 lines of apply helpers
  into `sim_core::engine`. A clean move remains possible later.
- **Bundle 3y**: events ticker. `EventsLog` Resource (VecDeque cap
  2000); `tick_sim` drains each tick's events Vec into the ring
  instead of dropping it. Panel renders newest-first with a kind
  filter and colour-coded kind cells.
- **Bundle 3z**: keyboard shortcuts. Spacebar toggles sim.running,
  Escape clears ship + facility selection. Both gated on
  `egui::Context::wants_keyboard_input` so typing in a filter box
  doesn't fire them.
- **Bundle 3aa**: admin Heal Ship. Green button in ship details
  refills fuel, resets morale / health to 100, clears hunger +
  fatigue, floors credits at 10k (never downgrades richer ships).
  Unsticks spiralling ships for debugging.
- **Bundle 3ab**: gate ring gizmos. Amber 48-segment Gizmos circle
  at each `facility_type == "gate"` body position, radius 0.7 wu.
  Visually separates jump gates from stations at a glance.
- **Bundle 3ac**: construction site gizmos. Outer amber scaffold
  ring (radius 0.9) + filled inner green progress ring whose
  radius scales with `site.progress()`. Gizmo-only so it handles
  dynamic spawn + materialization without a matching Bevy-entity
  lifecycle.
- **Bundle 3ad**: Reset view button + `CameraResetRequest`
  Resource. Top bar button flips the flag; the camera plugin's
  new `apply_reset_request` system restores `OrbitCamera` to
  `::initial()` pose. One-click recovery when pan/zoom drifts.
- **Bundle 3ae**: nebula backdrop — 14 translucent colored spheres
  at 9000-wu shell seeded off world_seed, palette biased toward
  purple/teal/dusty-orange. Past the star shell so they read as
  deep-background volumes. Alpha 0.04, emissive 0.10-0.22.
- **Visual polish pass (post-feedback)**: ClearColor black (was
  grey default), DirectionalLight `shadows_enabled: false` (fixed
  1-FPS symptom on M1 with 1604 shadow-casters), WinitSettings
  Continuous focused + unfocused (keeps window ticking when Terminal
  takes focus — prior ReactiveLowPower path showed as 1 FPS),
  camera PerspectiveProjection `far = 20_000` (default 1000 was
  clipping starfield + nebulae at galactic zoom), F12 screenshot
  via ScreenshotManager + `CLIENT_BEVY_AUTO_SCREENSHOT=1` env for
  headless capture at frame 3. BODY_SIZE 1→5 / FACILITY_SIZE
  0.35→1.5 / SHIP_SIZE 0.18→0.4 so agents are actually visible at
  galactic zoom. SELECT_SCALE 2.2×→4×. Faction ship materials carry
  emissive tint (1.8×); camera gets `hdr: true` + BloomSettings —
  ships now read as bright faction-coloured sparks at overview.
  `ClearColor(BLACK)` paired with ambient 80 (was 200) so ships
  don't wash out against space.
- **Bundle 3af**: pulsing cyan selection ring via Gizmos. Slow
  0.6 Hz sine, ±15% radius; bright enough to bloom through the
  HDR post-process so a selected ship stays findable at any
  zoom level.
- **Bundle 3ag**: Save-As DB backup. Top-bar "Save as…" button
  fires a VACUUM INTO snapshot of sim.db via a one-shot tokio rt
  + new `sim_db::persistence::save_as(pool, dest)` helper. Top
  bar split into two systems (`top_bar_controls` for pause /
  speed / tick / reset view / save; `top_bar` for panel
  toggles) because the combined param list hit Bevy's 16-param
  system-tuple limit.
- **Bundle 3ah**: admin Spawn Ship button. `sim_server::engine::
  admin_spawn_ship` promoted to `pub`; client_bevy's "+Freighter"
  button queues a `SpawnShipRequest` that `process_spawn_ship`
  consumes, building a payload (freighter at earth-station,
  independent, 5000 cr, timestamp-derived id) and mutating the
  hecs store. Paired `render::reconcile_new_ships` spawns the
  matching Bevy render entity on the next frame — mirror of
  `cleanup_stale_ships` for the birth side of the lifecycle.
- **Bundle 3ai**: docked ship list in facility details + cross-
  nav. Facility "Dock" subsection now lists each docked ship by
  name/class and each queue entry by position; clicking any row
  selects the ship (opens its details panel). Makes the dock
  FIFO + occupancy inspectable and tightens the facility→ship
  navigation path to match Godot.
- **Bundle 3aj**: admin Refill Reserves. Facility toolbar gains
  a button that floors every Commodity entry in
  `FacilityStorage.reserves` at 500 — never downgrades a richer
  stockpile. Quick way to recover a starving facility during
  testing without the per-commodity delta UI.
- **Bundle 3ak**: admin Refit combos. `admin_refit_ship` promoted
  to `pub`. Ship-details Fitting block becomes interactive — each
  slot row is a ComboBox of compatible module slugs (filtered via
  `sim_core::modules::modules_for_slot`). Selecting a different
  slug fires admin_refit_ship with `via_shipyard=false` (admin
  override, no cost or stock check). Derived stats refresh same
  frame.
- **Bundle 3al**: TPS readout in perf panel. Rolling
  `(Instant, tick)` ring → derived ticks/sec, coloured against
  the configured `8 × tick_mult` target. Diagnoses background-
  window throttling vs. real overload.
- **Bundle 3am**: tick compute p50/p99 in perf panel. New
  `TickPerf` resource records each FixedUpdate `tick_sim` wall-
  time into a 60-sample window; recomputes p50/p99 each tick.
  Two overlay rows show ms with budget-aware colouring (red
  approaches 125 ms FixedUpdate cadence). Mirrors
  `sim_server.perf.tick_compute_*`.
- **Bundle 3an**: ship-list hover tooltips. Each row's labels
  show "credits N / fuel X% / goal Y" on hover — quick-glance
  ship state without opening the details panel. ShipRow query
  expands to pull ShipEconomy + ShipFuel + ShipAI per row; cost
  is bounded by ship count which never blows up the UI loop.
- **Bundle 3ao**: sync decommission drain. tick_sim now drains
  `AgentStore.pending_decommissions` every tick — strikes
  accumulating to retirement actually retire. Mirror of
  `sim_server::drain_pending_decommissions` minus the persist_tx
  (no DB worker). Builds a RetirementRow snapshot, releases
  ship contract holdings, pops from facility docked + queue,
  despawns + removes from ship_by_id, prepends the row to the
  live `Retirements` resource (panel grows during play).
- **Bundle 3ap**: sync respawn drain. Pair to 3ao closing the
  death/rebirth loop. `drain_respawns` partitions
  `pending_respawns` by `spawn_at_tick`, fires due entries with
  deterministic seed-derived class / faction (70% prefer / 30%
  underdog) / dockable station, resolves body position from the
  in-memory hecs world (no DB JOIN), calls
  `sim_server::engine::admin_spawn_ship`. `random_ship_name`
  promoted to pub for reuse. Embedded sim now maintains a
  steady-state fleet under organic strike pressure.
- **Bundle 4y…4ai (visual + admin polish run)**:
  - **4y**: theme palette sweep across all panels (last hardcoded
    RGB ladders gone), construction site rings recolored green +
    crosshair tick-marks (was indistinguishable from amber gate
    rings), construction panel `f32::INFINITY` ProgressBar fix,
    procedural cuboid ships replaced with 5 authored Godot GLB
    hulls (Spitfire / Dispatcher / Imperial / Challenger /
    Executioner) via new `ShipModels` resource + load_ship_models,
    `SHIP_GLB_SCALE = 0.025`.
  - **4z**: top-bar hover tooltips on every panel-toggle, selection
    chip in `top_bar_controls` showing "ship · <id>" / "facility ·
    <id>" with ✕ Clear button.
  - **4aa**: Esc opens a centered Pause menu (Resume / Save As /
    Quit). Save-As fires `SaveAsRequest`; Quit sends `AppExit`.
    `PauseMenuState` carries `was_running` so restoring on close
    doesn't auto-resume an already-paused sim.
  - **4ab**: full Add Ship admin dialog — class / faction /
    location / credits / name dropdowns built from the live world.
    `SpawnShipRequest` evolves from `bool` to
    `Option<SpawnShipParams>`.
  - **4ac**: Add Facility admin dialog. `admin_spawn_facility`
    promoted to `pub` in sim_server. `reconcile_new_facilities` in
    render/ surfaces the new hecs entity as a SceneBundle on the
    next frame (analogue of `reconcile_new_ships`).
  - **4ad**: right-click context menu in the world (Inspect /
    Follow / POV / Destroy). New `CameraFollow` resource +
    `apply_camera_follow` system rewrite `OrbitCamera.focus` from
    the target's transform each frame; manual pan clears the
    follow lock. Ship/facility delete pipelines moved to
    `DeleteShipRequest` + `DeleteFacilityRequest` resources +
    `process_delete_*` systems.
  - **4ae**: per-commodity reserve editor inline in facility-
    details. -100 / +100 / +1k buttons per row + Add-commodity
    picker. Mutations buffered into `reserves_deltas` during the
    read-only render closure, applied after.
  - **4af**: bevy_hanabi engine particle trails. Single shared
    `EffectAsset` (cyan exhaust, 30 particles/s/ship, 0.6 s
    lifetime, 512 cap). Per-ship sibling trail entity tagged
    `EngineTrailFor(sim_id)`; `sync_engine_trails` glues each
    trail's transform to its ship's rotation + position with a
    1.5 wu aft offset.
  - **4ag**: trails gate on transit — `Visibility::Hidden` when
    `ShipPosition.docked_at.is_some()`, hanabi's default
    `WhenVisible` halts emission instantly so docked ships read as
    engines-off.
  - **4ah**: review polish — `+Fac` button uses correct
    `add_facility` icon; context-menu click-outside detection
    fixed (was using area-scoped `interact_pointer_pos` which
    returned None for outside clicks → menu never closed).
  - **4ai**: ship explosion particles on destroy / retire.
    `cleanup_stale_ships` spawns a one-shot
    `ParticleEffectBundle` (80 particles, 1.0 s lifetime,
    flash → orange → fade) at the doomed ship's last transform.
    `ExplosionLifetime` timer self-cleans 1.5 s later. Renders for
    every despawn path: admin Destroy, broke/hoard auto-retire,
    deletion via context menu, etc.

Deferred:
- Async drains (PendingMaterialization / PendingDecommission /
  PendingRespawn) — need tokio inside the fixed-update system.
- Persistence worker for the embedded client (flush dirty agents
  to sim.db so hours of play don't vanish on quit).
- Window remember-state — egui Memory persistence needs a
  bevy_egui hook + serde-bytes round-trip; risky without user
  verification.
- True 1st-person POV (4ad's POV menu item is a Follow at 80 wu
  rather than a camera attached to the ship's transform).

## Systems wired

| System | Where | Behavior |
|---|---|---|
| Ship BT planner | `agents/ship::plan_goal` | Picks goal + builds `BtNode<ShipAction>`: rest_crew → refuel → sell_cargo → fulfill_delivery / passenger / courier → trade_run → idle |
| Ship BT execution | `agents/ship::execute_ship_action` | Dispatch closure for ShipAction variants. Dock emits `RequestDock`; Refuel is multi-tick (drains facility fuel reserves); SellAll unloads one commodity per tick |
| Dock dwell | `engine::apply_request_dock` / `apply_facility_refuel` | Docking fee = base × occupancy × (2 − condition). Refuel: 5 units/tick × 2 cr/unit from `FacilityStorage.reserves[Fuel]` |
| Market tick | `agents/market::tick_markets` | 8 phases: produce → consume → emergency orders → surplus orders → match → update-demand → track-shortage → update-prices |
| Price engine | `economy/price_engine` | Coverage model (supply as buffer, not ratio) + shortage urgency multiplier |
| Facility recipes | `agents/facility::tick_recipes_and_mining` | 10 recipes, limiting-factor scaling, `max_batches` scales with facility level. XP grants per recipe executed |
| Mining | same | `mining_rig` facilities extract commodities from their body's resources |
| Facility condition | `agents/facility::tick_degrade_condition` | Condition drops 0.0002/tick × tick_scale; self-repair uses 1 Tool for +0.01 condition when <0.8 |
| Facility restock | `agents/facility::tick_restock` | Buys fuel/food/tools/water from local market when reserves < 100, targets 500 |
| Facility autonomy | `agents/facility::tick_post_contracts` | Posts delivery contract when reserves low AND local market short 30+ ticks |
| Facility abandonment | `agents/facility::tick_abandonment` | 200-tick countdown when condition<0.1 + no tools + credits<100. Abandoned → tick phases skip, services cleared in snapshot |
| Facility leveling | `agents/facility::grant_facility_xp` | Level 1–10, threshold = level×100. Higher level scales recipe max_batches |
| Population | `agents/population::tick_populations` | Consume food/fuel, produce labor, sentiment inertia (0.9^tick_scale), urgency-scaled food bid (1×→3× at sat=0), emergency water <30, post passenger contracts when pop≥5000 + sentiment≥50 |
| Contracts | `economy/contracts::ContractBoard` | 6 kinds: Delivery / Courier / Passenger / Subsidy / Supply / CrewMission. Posted → Accepted → Paid lifecycle, `faction_id` tag for rep routing, expiry sweep |
| Factions — tax | `agents/faction::run_tax_cycle` | Every 200 base-ticks, 0.5% tax on facility credits above 1000-credit buffer |
| Factions — subsidy | `agents/faction::run_subsidy_cycle` | Every 30 base-ticks, posts subsidy contracts for chronic member-market shortages. Multiplier 1.25×–3.0× ramps with shortage age. Max 3 active per faction |
| Crew | `agents/ship::tick_crew` | Hunger / fatigue / morale f32; `current_task` derived (Piloting/Engineering/Trading/Eating/Resting/Idle); eats food from cargo when hungry; wages paid every 50 base-ticks or morale drops |
| Crew skills | `agents/ship::ShipCrewSkills` | nav/eng/trade/combat u8 capped 100; grants +1 on Buy, +1 nav/trade on Deliver, cumulative toward a skill XP ceiling |
| Reputation | `ShipEconomy.faction_reputation` | +1 per contract delivered to tagged faction (0–100) |
| Body regen | `agents/celestial_body::tick_bodies_regen` | Mined resources regrow toward `max_resources` at 0.0005 × tick_scale per regen-cycle |
| Body biome | `BodyBiome.food_multiplier` | Maps biome name → 0.1..=1.5 food-availability multiplier; consumed by growth logic |
| Persistence | `db/persistence::PersistenceWorker` | mpsc buffer + dedup + 5s batch flush. Ships every tick, markets every tick, factions on change, facilities/populations every 16 ticks, bodies every 40 ticks, tick counter every 8, construction sites every 16. Load tick counter on startup → sim resumes from the right tick. 13 DbWrite variants today. Migrations 001–011 (see `crates/sim_server/migrations/`): ship/market/faction/facility/population tables, fitting + module_stock + ship_size + contract persistence, facility_crew (009), retirement_ledger (010), construction_sites (011). |
| Crew missions | `economy/missions` + `facility::tick_post_crew_missions` | 9 mission templates (port of Elixir `mission_catalog.ex`). Facility posts CrewMission kind (25% chance / cycle, 1 active max). Ship BT: `Dock → Refuel → Rest → Deliver → Undock`. Completion grants skill XP via `ShipCrewSkills::grant` |
| Contract rep gating | `ContractBoard::posted_for_ship` | Subsidy contracts start `min_reputation = 70` (faction-only), CrewMission starts 50 (allied), others 0 (public). Age-open decays 10 rep per 100 ticks → public after ~1000 ticks. Ship planner filters by `faction_reputation[fac] >= min_reputation` |
| Admin endpoints | `http::admin` | Full CRUD: **POST spawn** ships + facilities (INSERT + dispatch to engine), DELETE by id, market supply delta, save / save-as (VACUUM INTO) |
| Events broadcast | `engine` events Vec + `ws::protocol::Events` | ship_spawned, ship_docked, ship_decommissioned, facility_spawned, facility_removed, facility_abandoned, contract_delivered, crew_mission_completed, passengers_arrived, market_supply_adjusted — per-tick drain |
| Spatial LOD | `simulation::lod::AgentLod` + `engine::recompute_lod` | Observer-driven tiers 1/2/5/10/20/50. Ship tick phases gate on `active` map; fuel burn + crew hunger/fatigue scale by `skipped_ticks` so total state change over wall-clock matches. Recompute every 50 base-ticks. Markets never LOD'd. |
| Ship subsystems (18a) | `modules::CATALOG` + `ShipFitting` | 5 slots × 3 tiers (engine/cargo/sensors/reactor/comms). Tier-1 neutral. `ShipFitting::new_for_size` seeds size-aware defaults (size≥5 gets tier-2 sensors always; size=4 rolls 50% deterministic on ship id hash). Derived multipliers bake into `ShipCargo.capacity` + `ShipFuel.speed_modifier` at spawn/refit. |
| Ship refit (18b) | `http::admin::refit_ship` + `engine::admin_refit_ship` | `POST /api/admin/ships/:id/refit {slot, slug}` (admin override, no cost). Migration 003 persists `ships.fitting` as JSON; loader hydrates with fallback to tier-1 default. |
| Shipyards + module production (18c) | `economy::module_production` + `facility::tick_produce_modules` + `ModuleStock` | `facility_type = "shipyard"` mints modules from local market supply on a 15×mult cadence. 10 recipes (every non-tier-1 slug). Stock target 2/slug. Migration 004 persists `facilities.module_stock`. Admin refit with `via_shipyard: true` debits ship / credits yard / decrements stock. |
| Agent retrofit (18d) | `ShipProgression` + `find_retrofit_opportunity` + `SimCommand::RequestRefit` | Ships earn XP on delivery (15 xp each), level at `level × 100` threshold (cap 10), bank 1 retrofit point per level. Planner emits `seek_retrofit` when points>0 + credits>reserve + reachable shipyard with matching upgrade. BT: `DockAtShipyard → Dwell → Refit → Undock`. One point spent per install. Migration 005 persists progression. |
| Ship size (18f) | `simulation::config::ShipSize` + `ShipIdentity.size` | 6 tiers Tiny..Massive, orthogonal to class. Drives default sensor tier + sets up future speed/cargo differentiation. Class→default_size (courier=Small, freighter=Medium, tanker=Large, hauler=Huge, miner=Small). Migration 007. |
| Cross-system contract gate (18f) | `try_plan_contract` | Contracts spanning systems hidden from ships whose `sensor_range_au < 2.0`. Tier-1 sensors = intra-system only. Natural niche separation: big hulls with tier-2 default sensors see interstellar contracts day one. |
| Comms payment bump (18f) | engine `SimCommand::DeliverContract` apply | Delivered payment multiplied by `1.0 + ship.fitting.derived.comms_bonus`. Tier-2 comms = +5%, tier-3 = +10%. |
| Size speed penalty (18g) | `ShipSize::speed_multiplier` | Applied on top of `ShipClass.speed_modifier` + fitting at spawn. Tiny 1.15× → Massive 0.70×. Bigger = slower; amortises over larger cargo on long runs. |
| Facility subsystems (18e) | `facility_modules::FACILITY_CATALOG` + `FacilityFitting` | 2 slots × 3 tiers (production / automation). Production multiplies recipe output 1.0/1.15/1.30×; automation adds 0/1/2 flat batches to `level_scaled_max_batches`. Migration 006 persists. `POST /api/admin/facilities/:id/refit` (admin override only). |
| Personality planner (18i) | `plan_goal` + `score_goal` | Elective goal band (contract vs trade_run) scored by `ln(profit) × (1.0 + personality.prefer_goals[goal])`. Archetype `prefer_goals` entries (cautious_trader likes fulfill_delivery, aggressive_trader likes trade_run, courier likes passenger) now actually tilt the pick. Class→personality default via `Personality::for_class`. |
| Contract payment escalation (18j) | `ContractBoard::age_escalate_payment` | Posted contracts accrue +5% payment / 50 ticks, capped at +100%. Idempotent — recomputes from `initial_payment` each call. Hooked into population tick alongside `expire` + `age_open_reputation`. `FACILITY_DELIVERY_LIFESPAN_BASE` trimmed 500→300 so stale contracts churn + re-post at a fresh premium. |
| Rest timeout + non-docked recovery (18k) | `ShipAction::Rest` + `tick_crew` | `rest_start_tick: Option<u64>` on `ShipEconomy`. Rest succeeds on morale ≥ target OR after `MAX_REST_TICKS_BASE × mult = 300×` ticks. Non-docked `Resting` morale recovery bumped from 0.5× to 1.0× docked-baseline. Breaks the pin-at-zero death spiral when ships can't actually dock. |
| Multi-facility dock fallback (18k) | `apply_request_dock_typed` via `dockable_candidates_ordered` | When the primary (highest-cap) facility at a location is full, fall through to secondary / tertiary candidates. Haven-station's 5 facilities × 19 slots all get used now instead of 111 ships piling on the 8-slot station. |
| Dock queue awareness (18L) | `ShipAction::Dock` + `dock_attempt_start_tick` | Multi-tick: emit `RequestDock`, return Running until `docked_at` is set OR the context-dependent budget elapses. Has-contract = 150 base ticks (patient). Low morale + no contract = 30. Default = 60. On timeout, Failure → BT unwinds → planner replans. |
| Congestion-aware rest (18L) | `plan_goal` rest_crew branch + `pick_rest_destination` | `decide_and_act` builds per-tick `dock_congestion: HashMap<loc, (cap, docked)>` snapshot. When a ship's current location is saturated, `rest_crew` routes to the least-saturated same-system station instead of stampeding the current dock. |
| Facility crew (19a) | `FacilityCrew` + `tick_facility_crew` | Aggregate struct (not per-crew entities — that experiment made crew 60% of all agents in the Elixir days, off-table). `crew_count`, morale, hunger; food draw 0.02/crew/tick from reserves. `crew_efficiency` multiplier into recipe output (0.5 when starved, 1.0 when fed). Migration 009 persists `facilities.crew_state` JSON. |
| Contract auto-renewal (19b) | `ContractBoard::sweep_expiring` | Delivery + Construction renew up to `CONTRACT_MAX_RENEWALS=3` times, +25% payment, -15 min_rep per renewal. Subsidy gets ONE "desperate bounty" renewal that slams min_reputation to 0 — faction tried to keep the contract in-house first; if no loyal ship picked up, anyone can. |
| Demand analyzer + construction sites (19c) | `faction::run_construction_cycle` + `agents/construction` + `engine::materialize_facility` | Chronic shortage (≥ 600 ticks) posts a `ContractKind::Construction` with a spawned `ConstructionSite` entity at the destination. Ships delivering construction cargo bump `progress_delivered`; at 1.0 the site despawns and a real facility materialises at the same position. Migration 011 persists the sites; `PendingMaterialization` queue handles the DB INSERT + db_id backfill async after apply. |
| Population migration (19d) | `population::tick_populations` migration branch | `migration_pressure` accumulates when `overall_sentiment < 30`, decays above. Crossing threshold → pick highest-sentiment reachable pop as destination, move N people via `SimCommand::MigratePopulation`. Emits `population_migrated` event. |
| Reputation scaling (19e) | engine `SimCommand::DeliverContract` apply + breach on `ReleaseContract` | Delivery rep gain: `(5 + payment/2000).clamp(1,15)` — big contracts move rep faster. Breach penalty: `-(10 + payment/1000).clamp(-25,-1)` applied on dock-timeout release, taking real stakes for abandoning a run. |
| Retirement ledger (19g) | migration 010 + `retired_ships` table + `engine::admin_remove_ship` | Per-decommission INSERT of `{agent_id, name, class, faction_id, final_location, final_credits, final_crew_morale, final_fuel_pct, broke/hoard_strikes, retired_at_tick, reason}`. Table survives after the `ships` row is deleted so the Retired panel can scroll back through historical losses (live + DB-loaded). |
| Exploration + smart idle (19h) | `ship::smart_idle_fallback` + `try_build_explore_bt` | Post-plan idle_ticks counter increments when `current_goal.starts_with("idle")`. Past `IDLE_EXPLORE_TICKS * trigger_scale` and with credits/fuel OK, ship picks an adjacent system NOT in `ShipMemory.recent_systems` (VecDeque cap 5) and travels there. Anti-yo-yo; cross-system spread without pushing monoculture. |
| Adaptive fleet retirement (post-19) | `ship::decide_and_act` strike block + `drain_pending_decommissions` + `drain_pending_respawns` | `broke_strike` when credits<500 + cargo empty + no contract; `hoard_strike` when credits>200k + idle_ticks>500. 3 strikes of either → `SimCommand::DecommissionShip { reason }`. Async drains run admin_remove_ship, queue `PendingRespawn` 300-600 base ticks out; respawn mirrors admin-spawn INSERT with randomised class (courier/freighter/hauler/tanker/miner), faction (70% same as retired ship, 30% fewest-ship faction), location (faction-owned dockable station preferred), and a two-word name. |
| Dock queue (post-18L FIFO) | `FacilityStorage.dock_queue: Vec<Entity>` + `apply_request_dock_typed` | New arrivals admit immediately only when `docked + queue_len < capacity`; otherwise queue at shortest candidate facility's queue. Queue head gets admit priority. Broke heads self-evict on fee-reject (so they can't starve the line). Zombie sweep runs both inline (per `RequestDock`) and globally in `tick_facility_queue_sweep` every 10×mult ticks — drops entries whose ship no longer points at this location (typical cause: ship started a new Travel). Waiting ships pin to primary facility position so they orbit with their station. `FacilitySnapshot.dock_queue: Vec<String>` exposes the FIFO to wire-protocol clients. |
| Treasury tuning (Phase 19-tail) | `faction::*` constants | Money-laundering loop fix: bailouts used to pin every faction at $2k. New `FACTION_WORKING_CAPITAL_FLOOR = 10k` (shared by bailout + working capital), `FACTION_SUBSIDY_FLOOR = 5k` (lower so subsidy posts can fire in steady state). Tax rate 0.5% → 2.5%, bailout floor 3k → 500, bailout target 6k → 2k, construction 15k → 10k, subsidy base 2k → 800. Economy now sits $5.5k–$10k with subsidies still firing. |
| Subsidy source preference (Phase 19-tail) | `run_subsidy_cycle` | Three-tier picker: same-system non-member + supply>100 → any non-member with supply → max-supply anywhere. Pairs with the one-shot desperate-bounty renewal (19b) so subsidies default to intra-system loyal ships and open public only when local fails. |
| Admin delete cascade (post-19) | `ContractBoard::drop_ship_holdings` + `cancel_for_facility` + `admin_remove_facility` | Ship delete: Accepted held by this ship → Posted (re-postable); PickedUp → Expired. Facility delete: kick docked ships (`docked_at = None`), cancel contracts where this was requestor (always) + by source/dest only if last facility at the location, clear paired-gate back-reference. Events emitted per cascade step. |
| world_seed (post-19) | `db::persistence::load_or_init_world_seed` | u64 minted from SystemTime nanos on first boot, persisted through `simulation_state.world_seed`. Exposed on `/api/health.world_seed`, `/api/report.world_seed`, and the `server_hello` WS frame. `client_bevy` reads it directly from `Sim.world_seed` and seeds its deterministic nebula-cloud field off it so landmarks stay put across restarts. |
| Command-trace ring (20a) | `AppState.command_trace` + `engine::record_command_trace` | VecDeque<CommandTraceEntry> cap 1000 (~64 KB), written before every `apply_ship_commands` pass. `SimCommand::actor_ship` / `kind_label` / `trace_detail` helpers extract per-variant context. `GET /api/debug/agent/:id/trace?limit=N` returns oldest-first entries (sim_server). `client_bevy`'s ship-details panel reads the same ring directly from the `AppState` Resource. |
| Route-performance memory (20c) | `ShipMemory.route_performance` + `ShipCargo.cargo_origin` | Sell arm stamps origin + price on Buy, computes realised profit on Sell, records `RouteStats { attempts, total_profit_per_unit }`. `best_trade_route` multiplies raw margin by 0.3× when attempts≥3 AND avg_profit_per_unit < 50% of current projected gap. Nudges ships off saturated routes once the evidence accumulates. |

## Common tasks

| Task | Command |
|---|---|
| Compile + test + clippy | `scripts/verify.sh [--strict]` |
| Boot sim, inspect DB | `scripts/smoke.sh [secs]` |
| Boot + exercise HTTP | `scripts/ws-test.sh [secs]` |
| Verify persistence round-trip | `scripts/persist-test.sh` |
| Faster game-time for demos | `WORLD_SPEED=4.0 scripts/smoke.sh 30` |
| Trace sqlx queries | `RUST_LOG=trace scripts/smoke.sh` |

Default `WORLD_SPEED=0.25` (chill play). Earth-Mars ≈ 4 min. Good for
interactive play. For smoke tests / validation use `WORLD_SPEED=4` or
higher to see economic convergence in tens of seconds.

World speed in `client_bevy` is the top-bar combo (1× / 2× / 5× / 10×
labels mapping to internal `tick_mult` 1/2/5/10; the `tick_mult × 0.25`
scale anchored to the Godot WORLD_SPEED baseline isn't surfaced — clean
integers read better in-game). `tick_sim` runs the sim-tick body
`tick_mult` times per FixedUpdate, so 10× is genuinely 10× the agent
work per wall-second.

`sim_server` exposes the same knob over HTTP (`POST /api/sim/speed
{world_speed: N}`) for headless / scripted runs.

## Conventions

### Module documentation

Every non-trivial file starts with `//!` module-level docs explaining what
the module is responsible for and any non-obvious constraints. Examples:

- `agents/ship/mod.rs`: component split rationale + tick phase ordering; the module directory hosts `snapshot.rs` (wire types) and `actions.rs` (ShipAction enum + dispatch)
- `economy/price_engine.rs`: coverage model + urgency multiplier
- `economy/contracts.rs`: lifecycle + index structure
- `simulation/commands.rs`: why the command buffer exists
- `ai/behavior_tree.rs`: why BtNode is generic over A (fn-pointer capture problem)

When you add a file, document it the same way.

### hecs borrow patterns

`hecs::World::query_mut::<(&mut A, &mut B)>()` holds ONE mutable iterator
at a time. If a tick phase needs to read from markets while mutating ships,
structure it as two passes:

1. **Snapshot pass** — read-only query copies the needed data into a
   `HashMap<loc, snapshot>`
2. **Mutating pass** — iterate mutably, closing over the snapshot

Canonical patterns:
- `ship::tick_ships` — 3 passes: `advance_transits` → `observe_prices` → `decide_and_act`
- `facility::tick_facilities` — build location→market index, then iterate facilities
- `market::tick_markets` — collect entity list, then `query_one` per entity (single-entity multi-component mutable borrow)

### Commands vs direct mutation

- **Same-entity mutations** go in-place. A ship selling cargo updates its
  own credits + cargo directly.
- **Cross-entity effects** MUST go through `SimCommand`. Market supply
  deltas from ship trades emit `SimCommand::{Add,Remove}SupplyAtShipLocation`
  which the engine resolves to the destination market entity in a
  dedicated apply phase.
- Contract state transitions (Accept / Deliver) go via commands so the
  apply phase can pay the ship, mutate cargo, update market supply, and
  mark the contract Paid atomically.

### BT engine conventions

`BtNode<A>` is generic over a domain action type. For ships, A is
`ShipAction`. The tick method takes a dispatch closure:

```rust
let mut dispatch = |action: &ShipAction, bb: &mut Blackboard| -> BtStatus {
    execute_ship_action(action, entity, ..., bb)
};
bt.tick(&mut bb, &mut dispatch);
```

The closure owns the mutable borrows of the ship's components for this
ship's tick. When building a BT, bake parameters into the action variant
(e.g. `ShipAction::Travel { destination: "earth-station".into() }`) —
this is why `A` exists instead of plain `fn` pointers.

### Testing

- Pure logic tests live in `#[cfg(test)] mod` inside the module
- Integration smokes live in `scripts/` (bash)
- No test database is needed — the seeder is deterministic. Use `mktemp`
  for ephemeral test DBs.

### Don't do

- **Don't add `Mutex<AgentState>` per agent.** hecs owns the world; the
  tick loop owns hecs. Readers go through `SnapshotCache`.
- **Don't do async work inside the tick loop.** The tick is CPU work.
  For I/O, enqueue a command and handle it in a tokio task (the
  persistence worker is the canonical example).
- **Don't serialize whole agent state to JSON per tick.** Snapshot types
  are trimmed for a reason — they're what `sim_server` actually
  broadcasts. `client_bevy` reads hecs directly so it can splurge on
  detail in panels without paying serialization costs every tick.
- **Don't bring in Actix, bevy_ecs (vs hecs), or other heavy frameworks**
  without a clear justification documented in the commit message.
- **Don't call `query_mut` while holding a `query_one` result** on
  another entity — hecs only allows one mutable iterator at a time.
- **Don't write directly to a component across entities** mid-phase. Use
  a `SimCommand` so the apply phase does it after the iteration.

## sim_server protocol (headless / CI only)

`client_bevy` does NOT use this — it reads the hecs `World` directly
through the `Sim` Resource. The protocol is kept alive for the
`sim_server` binary, which still serves the smoke / persistence /
WS-roundtrip scripts under `scripts/` and remains buildable as a
standalone headless harness. Plain JSON over WebSocket on port 8080;
canonical reference is `crates/sim_server/src/ws/protocol.rs`.

**Server → client** (pushed):
- `server_hello` — on connect: tick + running + `world_speed` + interval + `world_seed`
- `positions` — every tick (125 ms). `type: "ship" | "body" | "facility"`
- `ship_states` — every 8 ticks (~1 s). Full snapshots (incl. `crew.current_task`, `crew.skills`, `crew.hunger`)
- `market_states` — every 16 ticks
- `facility_states` — every 16 ticks. Carries `docked_ships`, `dock_queue`, reserves, `level`, `xp`, `abandoned`
- `population_states` — every 16 ticks
- `faction_states` — every 32 ticks
- `contract_states` — every 32 ticks (6 kinds × 6 statuses, `min_reputation`, `faction_id`, `mission_category`)
- `events` — every tick when non-empty. `ship_docked`, `contract_delivered`, `ship_decommissioned`, `ship_leveled_up`, `ship_refitted`, `facility_refitted`, `passengers_arrived`, `market_supply_adjusted`, etc.
- `agent_state` — reply to `get_agent` / `get_snapshot`
- `sim_control_ack` — reply to pause/resume/speed

**Client → server**: `hello`, `get_agent`, `get_snapshot`, `move_observer`, `sim_control`

HTTP (same port, `/api/*`):
- Read: `/api/health` (+ `perf.tick_compute_p50_us` / `p99_us` + `world_seed`), `/api/report`, `/api/{ships,markets,facilities,populations,factions,positions,contracts,observers,construction_sites,retirements}`, `/api/agent/:id`
- Debug: `GET /api/debug/agent/:id/trace?limit=N` — last N ship-originated SimCommands, oldest-first, clamp 1..=200
- Control: `POST /api/sim/{pause,resume,speed,step}` — step takes `{ticks: u32}` and forces N ticks through even when paused
- Admin: `POST /api/admin/ships`, `DELETE /api/admin/ships/:id`, `POST /api/admin/ships/:id/refit`, `POST /api/admin/ships/:id/grant-xp`, `POST /api/admin/facilities`, `DELETE /api/admin/facilities/:id`, `POST /api/admin/facilities/:id/refit`, `POST /api/admin/markets/:loc/supply`, `POST /api/admin/save`, `POST /api/admin/save-as`

`client_bevy` mirrors every admin endpoint as a direct in-process call:
context-menu Destroy → `DeleteShipRequest` resource → next-frame
processor calls `sim_server::engine::admin_remove_ship`. Add Ship
dialog → `SpawnShipRequest` → `process_spawn_ship` →
`sim_server::engine::admin_spawn_ship`. Refit combos in ship details
→ `admin_refit_ship`. No HTTP, no JSON; the apply helpers (`pub fn`
in `sim_server::engine`) are shared between both binaries via the
`[lib]` target.

Admin delete paths cascade identically in both clients: ship delete
releases held contracts (`drop_ship_holdings`); facility delete kicks
docked ships, cancels tied contracts, clears paired gate back-references
(`cancel_for_facility`, `admin_remove_facility`).

## Running the binaries

### `client_bevy` (canonical client)

```
make run                # cargo run -p client_bevy (debug)
make build              # cargo build --release -p client_bevy
```

Saves live next to the binary as `<universe-name>.db` (default `sim.db`).
The Bevy app boots into the main menu; Start opens the universe-name
modal (default `sim`, collisions auto-suffix), Load Game lists every
`*.db` in the binary directory with the active session pinned.

For dev: assets resolve via Cargo's path so editing
`crates/client_bevy/assets/...` is hot-pickable on next run. For packaged
builds, `make package-mac` etc. copy `assets/` next to the binary so
Bevy's default `AssetPlugin` lookup works without an override.

### `sim_server` (headless)

```
cargo run --bin simulator
# stdout: sim:ready ws=ws://127.0.0.1:8080/ws http=http://127.0.0.1:8080/api tick=0
```

Persistence worker has up to 1000 ms on SIGTERM / SIGINT to flush its
final batch before exit. Smoke + perf scripts in `scripts/` exercise
this path; CI uses it for round-trip persistence verification.

## Inspector panels (Bevy / egui)

All panels live in `crates/client_bevy/src/ui/mod.rs`. They share the
`OpenPanelStack` (Esc closes top-most) and use the cyan / amber / green
/ red theme palette from `ui::theme`. Every panel reads from the `Sim`
Resource directly — the inspector IS a live debugger with full
component access.

| Panel | Toggle | Role |
|---|---|---|
| Perf overlay | top-bar Perf | FPS, ticks/sec vs. target (8 × tick_mult), p50/p99 tick compute ms, phase-cost breakdown |
| Ship list | top-bar Ships | Sortable / filterable roster, hover tooltips (credits / fuel / goal), click → details |
| Contracts | top-bar Contracts | All 6 kinds × 6 statuses, age + payment sort, show-expired toggle, paged |
| Factions | top-bar Factions | Ledger (ship/fac counts, treasury, active subsidies); pairwise relations matrix |
| Markets | top-bar Markets | Per-location collapsing headers; commodity rows with price / supply / demand / crisis flags |
| Pops | top-bar Pops | Census — count / sentiment / food / wealth / migration; coloured sentiment + food bands |
| Facilities | top-bar Facilities | Stations + refineries + factories + gates + shipyards; abandoned strike-through; click → details |
| Retired | top-bar Retired | Decommissioned ledger (broke / hoard / admin); reason + final credits + final crew morale + retired-at-tick |
| Construction | top-bar Construction | Active sites with ProgressBar, sorted by progress desc; per-card Focus camera button |
| Events | top-bar Events | Rolling tick-event stream with kind filter, colour-coded kind cells |
| Map | top-bar Map | Top-down minimap; bounds auto-fit each frame; click anywhere → recenter camera |
| Labels | top-bar Labels | Body name labels overlaid in screen-space (toggleable to declutter) |
| Ship details | click ship | Full inspector — every component (identity, economy, fitting, fuel, AI, crew, progression, memory, recent commands trail). Admin row: Heal / +XP / Destroy / Refit combos |
| Facility details | click facility | Identity + ops + reserves + crew + fitting + docked-ships list + queue. Admin: Refill reserves / Refit / Destroy / per-commodity supply delta |
| Pause menu | Esc / `P` / Menu button | Resume / Save As / Options / Help / Return to Main Menu / Quit |
| Options | pause → Options | Audio sliders (music + SFX); applied live |
| Save list | main menu Load Game | Active `sim.db` pinned; snapshot rows with Load + Delete (sidecar WAL/SHM cleanup) |
| New universe | main menu Start | Sanitised name field + collision-resolved filename preview |
| Add ship | top-bar +Ship | Class / faction / location / credits / name dropdowns from live world → in-process `admin_spawn_ship` |
| Add facility | top-bar +Fac | Name / type / body / faction / docking_capacity / credits → in-process `admin_spawn_facility` |
| Help | `?` / `H` / pause → Help | Hotkey reference (Space / Esc / P / WASD / scroll / right-click / F12) |
| Context menu | right-click world | Inspect / Follow / POV / Destroy on hit ship or facility |

World gizmos + overlays:
- Selection ring (cyan, 0.6 Hz pulse) on the selected ship or facility
- Gate rings (amber, 48-segment) at every `facility_type == "gate"` body
- Construction-site rings (outer amber scaffold + inner green progress) sized by `site.progress()`
- Engine-trail particles (cyan exhaust, hidden when docked)
- Explosion particles on every despawn path
- Transit-line gizmos from each in-transit ship's `departure_position` to its current position
- Camera reset button + `CameraResetRequest` snaps `OrbitCamera::initial()`

Theme module: `ui::theme` exports `PRIMARY` cyan, `SUCCESS` green,
`WARN` amber, `DANGER` red, `TEXT_DIM`, `CHROME_BG` (top-bar), and
helpers `section_header()`, `caption()`, `crew_bar_color()`. Every
panel should pull colours from there.

## Things intentionally left out (not critical)

- **Facility + population LOD rate-scaling** — Phase 15 wired ship LOD
  (where 75% of tick cost lives). Facilities + pops carry `AgentLod` so
  the perf HUD sees them, but their per-tick phases don't gate on it
  yet. Phase 16 candidate if a 10k-agent push shows them as a
  bottleneck. Recipe `max_batches` and sentiment-inertia math would
  need careful scaling.
- **Full per-tick command log / replay** — we have the live-ring
  version (Phase 20a, see Systems wired) which covers the common
  "what did ship X do the last N commands" debug case. A persistent
  replay log (tick-by-tick append to disk for post-mortem of a
  multi-hour sim) is a bigger lift and unbuilt; architecture supports
  it since commands are plain data.
- **Rayon parallelism** — dependency is in, never called. Architecture
  is designed for it (no cross-ship writes within a phase). Skip until
  10k+ agents. Current 800-agent tick is ~0.5–2 ms.
- **Windows native build validation** — cross-compile setup exists
  (`rustup target add x86_64-pc-windows-gnu` + bundled SQLite so no
  DLL). User will test on their Windows box.
- **Per-crew-member simulation** — crew is aggregated on the ship.
  800 ships × avg 4 crew = 3200 individual entities we're NOT spawning,
  which was an explicit call. Individual variation is in the SQLite
  `crew_members` table for future richer missions. Skills + tasks live
  at aggregate level via `ShipCrew` + `ShipCrewSkills`.

---
> Source: [Kalcode/spaceprojectsim](https://github.com/Kalcode/spaceprojectsim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
