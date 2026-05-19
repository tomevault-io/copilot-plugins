## dice-gunmen

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Godot 4.6 turn-based 2D PvP shooter. Main scene is [src/level.tscn](src/level.tscn). See [README.md](README.md) for full game rules.

## Commands

```bash
# Run the project
godot --path .

# Validate a GDScript file
godot --path . --check-only --script src/<file>.gd --headless --quit

# Re-import resources headlessly (useful after editing .tres / adding assets)
godot --path . --import --headless --quit
```

There is no test suite, lint config, or build pipeline. The project is run and iterated through the editor.

## Architecture

### Turn-state orchestration

[src/level.gd](src/level.gd) owns the game's state machine (`INITIAL_ROLL` → `TURN_INTERMISSION` → `TURN_ACTIVE` → `GAME_OVER`). It does **not** poll players — instead it pushes control via `player.set_controllable(true/false)` and listens for `action_performed` and `died` signals coming back from the players. AP is owned by the level and decremented when an `action_performed` fires; the player never sees the AP count. This means a new "action" type (e.g. push, reload) just needs to emit `action_performed` after succeeding to plug into the turn loop — no level-side code change required.

### Two grids, queried independently

There are two grids in play and they are deliberately not aligned:

- **Player grid**: `GRID_SIZE = 60` px, declared as a `const` in [src/player.gd](src/player.gd), [src/box.gd](src/box.gd), and [src/door.gd](src/door.gd). All gameplay positioning (player snap, box cells, door edge midpoints) uses this.
- **Tilemap grid**: the TileSet's intrinsic cell size (16 px by default in [assets/tileset.tres](assets/tileset.tres)). Used only for static terrain — Floor and Obstacles TileMapLayers in [src/level.tscn](src/level.tscn).

When checking walkability the code converts the player-grid world position into the tilemap's cell coords via `layer.local_to_map(layer.to_local(world_pos))`, so the mismatch is transparent at the API level — but a single 60-px player step can visually jump over a thin tilemap obstacle. Worth knowing before painting more levels.

### Group-based discovery, no manager

Cross-scene wiring uses Godot groups, not direct references or a singleton:

- `floor_layer`, `obstacle_layer` — the two TileMapLayers in level.tscn
- `box`, `door` — Cover objects placed in level.tscn

Players, boxes, and doors all do `get_tree().get_first_node_in_group(...)` or `get_nodes_in_group(...)` for lookups. This keeps `player.gd` and the cover scripts agnostic to where exactly things sit in the scene tree. Group membership is declared on the node in the .tscn (e.g. `groups=["box"]`); covers also re-add themselves in `_ready` as a safety net.

### Movement check ordering

`Player._try_move` ([src/player.gd](src/player.gd)) runs three checks in order, each of which can short-circuit:

1. **Door** on the edge between `current_cell` and `target_cell` — if present, `door.try_pass()` rotates ±90° from its current state; out-of-range (would exceed ±90°) returns false.
2. **Box** at `target_cell` — if present, `box.try_push(direction)` recurses through any chain of boxes; if any link is blocked by wall/obstacle/door/edge-of-floor, the whole chain refuses.
3. **Floor + Obstacles** — destination cell must have a Floor tile and no Obstacles tile.

A failed check returns early **without emitting `action_performed`**, so failed moves don't consume AP.

### Damage as duck typing

`Player._fire_weapon` uses `if collider.has_method("take_damage")` rather than type checks. Any new damageable entity (different cover type, destructible terrain) only needs:

- a `StaticBody2D` (or other PhysicsBody) on collision layer 1 so the player's `RayCast2D` hits it
- a `take_damage(amount: int)` method

No player-side branching needed. The default collision layer/mask = 1 matches the tileset's `physics_layer_0/collision_layer = 1`, so closed doors, boxes, and obstacle tiles all block shots out of the box.

### Door axis from position parity

[src/door.gd](src/door.gd) does not use exports for axis or rotation — both are derived from the node's transform at `_ready`:

- If `position.x mod GRID_SIZE ≈ GRID_SIZE/2`, the door is on a **vertical edge** (between two horizontally-adjacent cells). Closed-orientation rotation is 90°.
- If `position.y mod GRID_SIZE ≈ GRID_SIZE/2`, the door is on a **horizontal edge**. Closed-orientation rotation is 0°.

Editor `rotation_degrees` is read on `_ready`, normalized to `[-180, 180]`, clamped to `[-90, 90]`, snapped to the nearest of `{-90, 0, +90}`, and stored as `rotation_state`. So placing a door is purely a transform operation — no inspector fields to set.

## Godot file format gotchas

When editing `.tscn` / `.tres` directly (which is sometimes faster than the editor for adding groups, refs, or simple property tweaks):

- Use `ExtResource("id")`, never `preload()`.
- No GDScript keywords (`var`, `const`, `func`) inside resource files.
- Typed arrays must be `Array[Type]([...])`, not plain `[...]`.
- Group membership goes on the node header line: `[node name="X" type="Y" parent="." groups=["group_name"]]`.

For complex scene-graph edits, prefer the editor — text-editing instanced child nodes is fragile.

---
> Source: [duckenclaw/dice-gunmen](https://github.com/duckenclaw/dice-gunmen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
