## alice-gameengine

> Rust game engine. 40 modules, 787 tests. See `LLM_REFERENCE.md` for full API routing.

# ALICE-GameEngine — Gemini Reference

Rust game engine. 40 modules, 787 tests. See `LLM_REFERENCE.md` for full API routing.

## Build

```bash
cargo test --features full
cargo run --example spinning_cube --features full
cargo run --bin mcp_server  # MCP JSON-RPC server
```

## Quick game

```rust
use alice_game_engine::easy::*;
let mut game = GameBuilder::new("Demo").build();
game.add_camera();
game.add_cube(0.0, 1.0, -5.0);
game.add_light(0.0, 10.0, 0.0);
game.run_headless(300);
```

## Module map

Core: `ecs` (SoA), `scene_graph` (mesh+SDF), `math`, `engine`
Rendering: `renderer`, `gpu`, `shader`, `texture`, `lod`
Physics: `physics3d` (Verlet, SDF CCD), `collision` (GJK)
Audio: `audio` (HRTF, bus, PCM, spatial)
Gameplay: `ability` (GAS), `animation`, `navmesh`, `scripting`, `llm` (NPC AI), `verse` (UE6)
2D: `scene2d` (Sprite, TileMap)
Integration: `bridge` (ALICE-xxx traits), `mcp` (JSON-RPC), `import` (Unity/UE5)
Assets: `sdf_assets` (991 free .asdf.json from Open-Source-SDF-Assets), `asset` (OBJ/glTF)
SIMD: `simd_eval` (8-wide f32x8), `fix128` (128-bit precision)
Convenience: `easy` (5-line API), `prelude` (single import)

## Verse (UE6) primitives

- `Failable<T>` / `decide()` — failure context
- `Transaction::execute()` — rollback on failure
- `LiveVar<T>` — reactive dirty-tracking variable
- `StickyEvent<T>` / `SubscribableEvent<T>` — typed events
- `Coroutine` + `TickExecutor` — tick-aligned cooperative tasks

## SDF Assets (991 free models)

```rust
use alice_game_engine::sdf_assets::*;
let asdf = load_asdf_file("collections/pm-momuspark/Bench_01_Art.asdf.json").unwrap();
add_asdf_to_scene(&mut scene, "bench", &asdf, Vec3::ZERO);
```

Source: <https://github.com/ext-sakamoro/Open-Source-SDF-Assets>

## MCP (AI agent control)

```bash
claude mcp add --transport stdio alice-engine -- cargo run --bin mcp_server
```

Tools: `scene_list`, `scene_add_node`, `scene_remove_node`, `scene_set_transform`, `engine_status`, `physics_step`

## NPC AI

```rust
use alice_game_engine::llm::*;
let llm = MockLlm::new("Hello traveler.");
let mut npc = NpcContext::new("Guard", "stern");
let reply = npc.respond("Who are you?", &llm).unwrap();
```

## Notes

- `--features full` for GPU/audio/SDF
- `ComponentStore<T>` needs `T: Clone`
- Physics uses Verlet integration

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ext-sakamoro)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ext-sakamoro)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
