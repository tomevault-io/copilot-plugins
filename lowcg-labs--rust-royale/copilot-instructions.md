## rust-royale

> The core physics simulator and combat loop are fully operational.

# Rust Royale — Development Roadmap

---

## ✅ Phase 1: The Foundation & Combat (COMPLETED)

The core physics simulator and combat loop are fully operational.

- **The Data Layer:** Strict Enums (`SpeedTier`, `TargetPreference`, `SplashType`, `SpellType`) and JSON parsing via `serde` so the game never crashes on bad data.
- **The Architecture:** A pure `lib.rs` engine with a dedicated `sandbox.rs` test harness. The sandbox has infinite elixir and dual-click spawning (Left = Blue, Right = Red).
- **The Economy:** Real-time Elixir generation (1 per 2.8s), on-screen UI text, 10.0 cap, and cost validation that rejects spawns when the player is broke.
- **The Combat Loop:**
  - Euclidean distance targeting (`targeting_system`) that scans enemies within `attack_stats.range`.
  - Timer-based melee damage (`combat_damage_system`) with entity despawning on kill.
  - Ghost Target Fix: Instant dead-target clearing so units never freeze waiting for a corpse.

---

## ✅ Phase 2: Entity Physics & Polish (COMPLETED)

Units now feel like real objects being deployed onto the battlefield, not ghosts that instantly react.

- **Deployment Delay:** `DeployTimer` component (read from `deploy_time_sec` in JSON). Troops stand completely still for 1.0 second to "load into" the arena. The `Without<DeployTimer>` filter on `targeting_system` and `physics_movement_system` ensures deploying units are invisible to the combat AI.
- **First Attack Timing (Quick Draw):** `first_attack_sec` field in JSON + `AttackStats`. When a troop locks onto a target, the `AttackTimer` is dynamically shrunk to 0.5s for the rapid first strike. After that first swing, the timer is reset to the normal `hit_speed_ms` (1.2s for Knight) for all subsequent attacks. This matches real Clash Royale's "pre-charge" mechanic.

### 🔲 Still TODO in this phase:
- **The Funnel Problem (Local Avoidance):** Writing a `troop_collision_system` that reads the `mass` and `footprint` of units. This applies push-based separation physics so troops shove each other apart (heavier units displace lighter ones) and don't overlap into a single "Mega-Knight."
- **Same-team stacking prevention:** Clicking the same tile 5 times currently spawns 5 knights on the exact same pixel.
- **Flying units bypass ground collision:** `is_flying: true` exists in JSON but nothing in the engine respects it yet.

---

## 🟡 Phase 2.5: Air vs Ground Targeting (NEW — CURRENT FOCUS)

This should be done before pathfinding. Every system built after this point will assume correct air/ground filtering.

- **Targeting Filters:** Update `targeting_system` to check `targets_air` / `targets_ground` on the attacker and `is_flying` on the defender. A Knight (`targets_air: false`) should completely ignore a Baby Dragon.
- **Target Preference:** Troops with `target_preference: Buildings` (like a future Giant) should ignore all enemy troops and walk straight to towers.
- **Add `is_flying` and `targets_air`/`targets_ground` to the spawned ECS components** so the targeting query can filter on them.

---

## 🟠 Phase 3: The Arena & Navigation (The Map)

Once troops have physical mass and correct targeting, we must build the map they walk on. Split into two sub-phases to avoid A* blocking progress.

### Phase 3A: Static Obstacles & Wall Avoidance
- **Tower Entities:** Promote towers from painted tiles to real ECS entities with `Health`, `AttackStats`, `Team`, and `Position`. This lets the combat system attack towers automatically, and troops with `target_preference: Buildings` can find them via the existing targeting query.
- **Tile Collision:** `physics_movement_system` needs a "don't walk into walls" check — stop troops from entering River or Tower tiles.

### Phase 3B: Pathfinding
- **A* Navigation:** Replace straight-line movement with a node-based pathfinding algorithm so troops walk around the river and queue up naturally at the two bridges.
- **Bridge Queueing:** Depends on the collision system from Phase 2 being functional, otherwise troops clip through each other at chokepoints.

---

## 🔵 Phase 4: Match Logic & Game Rules (The Game)

Turning the map into an actual 3-minute game.

- **Dual Player Economies:** Currently only one `PlayerState` exists. Player 2 needs their own elixir bank and card hand.
- **Match Clock & Phases:** A global timer that handles the 3-minute match, triggers 2x Elixir at the 2:00 mark, and initiates Overtime.
- **Win Condition:** Halting the game, freezing all troops, and declaring a winner when a King Tower falls or the clock runs out.
- **Crown Counting:** Track how many towers each player has lost. If the clock expires, the player with more crowns wins. If tied → Overtime → Tiebreaker.
- **Overtime Rules:** First tower kill wins, elixir generates at 2x the entire overtime period.

---

## 🟣 Phase 5: The Deck & Card System (The UI)

Removing the sandbox "god mode" and adding the real constraints.

- **The Queue Array:** Implementing a strict 8-slot circular buffer to hold the deck. Initial 4 cards are random, but after that the rotation is deterministic (you always know which card is "next").
- **The Hand:** Drawing 4 cards into an active UI hand at the bottom of the screen.
- **The Rotation:** Shifting the buffer and pulling a new card into the hand when Elixir is spent.
- **Elixir Overflow Indicator:** Visual feedback (pulsing bar) when the player is at 10.0 and wasting generation.

---

## ⚫ Phase 6: Advanced Engine Mechanics (Optimization)

Scaling the engine to handle chaos without dropping frames.

- **The Spatial Hash Map ("Global Spacing"):** Upgrading combat to use grid "buckets." Instead of O(N²) math, troops only scan the exact tiles next to them. This lets the engine handle 100+ units at 60 FPS.
- **Re-targeting After Kill:** When a knight kills its target, it should immediately re-scan for the next closest enemy *that same frame* instead of walking for one frame and then re-acquiring (eliminates the visible "stutter step").
- **Splash Damage (Troops):** Valkyrie's `splash_radius` + `splash_type: self_centered` needs to hit all enemies in a radius around her, not just her target.
- **Spells & AoE:** Handling cards like the Fireball that don't walk, but instead spawn, deal radius damage, and instantly despawn.
- **Ranged Projectiles:** Princess Tower arrows and similar ranged attacks need a *projectile entity* that takes time to travel, rather than instant damage on timer tick.
- **Death Spawns:** `death_payload_id` in `BuildingStats` for units like Golem splitting into Golemites on death.
- **Dynamic Deployment Zones:** Repainting the red/blue grid validation when an enemy Princess Tower is destroyed so you can drop troops further up the map.

---

## ⚪ Phase 7: Multiplayer (1v1)

Taking the game online.

- **Fixed Timestep First:** Migrate all game logic from `Update` to Bevy's `FixedUpdate` schedule. `time.delta_seconds()` is inherently non-deterministic across two machines. A fixed timestep ensures identical simulation results given identical inputs.
- **Rollback Netcode:** Integrating `ggrs` to sync Player 1 and Player 2 inputs across the internet deterministically.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lowCG-Labs)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lowCG-Labs)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
