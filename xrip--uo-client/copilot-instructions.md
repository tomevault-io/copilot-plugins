## uo-client

> IMPORTANT: While user talk to you in Russian you should talk back to user using only English

IMPORTANT: While user talk to you in Russian you should talk back to user using only English

# Repository Guidelines

## Project Structure & Module Organization

This is a C++17 CMake project for an Ultima Online client and tooling. Core code lives in `src/`: `net` handles sockets, packets, and Huffman; `mul` loads MUL/verdata assets; `render` draws through MiniFB; `bot` contains pathfinding/blacklist logic; `builders` creates packet payloads. Public headers are under `include/uo`, with MiniFB in `include/win32`. Probes live in `tests/`. Generated outputs belong in `build/` or `cmake-build-*`.

## Client Runtime Notes

`Client` owns protocol state, packet dispatch, movement, bot commands, renderer ticks, and cached objects. Bot movement is pipelined (`kMaxInFlight = 4`, a fastwalk stack): several `0x02` moves may be in flight, each predicting position locally and reconciling on `0x22` ack or snapping back on `0x21` reject. The reject carries the authoritative pose, and a block denies every queued move behind it, so `OnMoveReject` de-dups the resulting redundant rejects and restores speculatively-consumed path steps. `bot::FindPath` runs A* over `World::QueryCell`; `blacklist_`, cached mobiles, and dynamic `0x1A` items add obstacles. Lookahead patches short prefixes of `botPath_`; full `BotReplanToGoal()` is the fallback.

`render::Minimap` overlays an orientation panel onto the world frame in `RenderTick` (top-right corner, toggle with `M`, composited via `Renderer::Overlay`). It uses the same isometric projection as the 3D view (`su = x-y`, `sv = x+y`), so north points to the top-right corner. Cell colours come from `radarcol.mul` (`render::RadarColors`) using the real client's radar rule — the topmost surface wins (land, then statics by z-test) — modeled on `CRadarGump_Update`/`CRadarGump_RenderMinimap` in `client_2.0.7.exe`. Colours are cached per 8x8 map block. The view auto-scales (in iso screen space) so the player and the whole planned `botPath_` route fit, and draws the route plus player/goal markers. The panel is only drawn when `radarcol.mul` loads (`cfg_.radarcolPath`).

When reverse-engineering with IDA, always save newly confirmed behavior and corrections to bad decompiler guesses as IDA comments or names, then save the IDB. Rename variables, params and functions if that needed. The local source should not become the only record of client-2.0.7 findings.

## Build, Test, and Development Commands

- `scripts\build.bat`: configures Ninja with Visual Studio Build Tools and builds all CMake targets.
- `cmake -S . -B build -G Ninja` then `cmake --build build`: manual configure/build when the compiler environment is active.
- `scripts\build_hufftest.bat`: builds and runs the Huffman round-trip test.
- `scripts\build_bltest.bat`: builds and runs the blacklist round-trip test.
- `scripts\build_pathprobe.bat [args]`: builds and runs the pathfinding probe.
- `scripts\build_viewer.bat [args]`: builds and runs the world viewer probe.
- `scripts\render_regression.bat`: Windows-only renderer regression harness; dumps PNG scenes to `build\regression\` for visual comparison with the official 2.0.7 client. Do not use Bash/WSL here.

## Coding Style & Naming Conventions

Use C++17 unless a probe requires newer syntax. Keep code exception-free and RTTI-free to match CMake flags. Follow the existing style: 4-space indentation, same-line braces, `uo` namespace, PascalCase classes, and lowerCamelCase fields such as `loginHost`. Prefer aliases from `include/uo/types.h` for protocol/file data.

## Testing Guidelines

Place focused probes in `tests/` with behavior-oriented names, such as `huffman_roundtrip.cpp` or `path_probe.cpp`. For pathfinding changes, run `scripts\build_pathprobe.bat sx sy sz gx gy [margin]` on a real MUL route. New coverage should get a script under `scripts/` or a CMake/CTest target.

After ANY change under `src\bot\` (A*, blacklist, lookahead) or to `World::QueryCell`/walkability, you MUST run `scripts\path_regression.bat` and diff the regenerated `tests\path_regression.txt` against the committed baseline. It runs two long cross-continent routes (Trinsic bridge <-> Britain basement, both directions). Treat `result` / `steps` / `expanded` / `pathCost` as deterministic regression signals — a diff there is a real behavior change and must be explained; `searchUs` is wall-clock timing (machine/run dependent), so read it as a performance trend, not a hard check. Commit the updated baseline only when the behavior change is intended.

For renderer changes, run `scripts\render_regression.bat` and inspect all generated PNGs, especially `07_negz_interior.png` for negative-Z interiors. The harness uses `scripts\build_viewer.bat` and writes only under `build\regression\`.

## Commit & Pull Request Guidelines

Commits use concise, imperative, scope-prefixed messages, for example `bot: add path lookahead rerouting`. Keep commits focused and use scopes like `net:`, `mul:`, `render:`, `bot:`, or `tests:`. PRs should describe behavior, list commands, mention `*.mul` assets, and include screenshots for renderer-visible changes.

## Security & Configuration Tips

Do not commit credentials, shard-specific secrets, or private asset paths. `src/main.cpp` currently contains local defaults for host, login, and MUL asset locations; prefer command-line overrides or local-only edits when testing against private environments.


@bot-client.md

---
> Source: [xrip/uo-client](https://github.com/xrip/uo-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
