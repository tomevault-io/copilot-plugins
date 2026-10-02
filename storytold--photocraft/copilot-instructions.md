## photocraft

> PhotoCraft is an open-source, native, Photoshop-comparable image editor written in **Rust only** (no JavaScript or TypeScript). The product name is always written **PhotoCraft** (`{Function}Craft` in PascalCase, like its siblings ArtCraft, ArtCraftX, DesignCraft, DrawCraft, EffectCraft, FilmCraft, LightCraft, PrintCraft) in user-facing text: UI, window titles, About, installers, release names, docs prose. Machine names stay lowercase: crates (`photocraft-*`), binaries, file names, ids (`ai.storyteller.photocraft`). Standards and learnings shared across the crafting apps live in `../craftrules` (read its `README.md`). Contribute reusable learnings there, never code; repos don't share code. The goal is 1:1 Photoshop parity (same menus, shortcuts, behaviour and file fidelity) with better performance, and every feature drivable by agents. Read this file first, then `docs/`.

# AGENTS.md: guide for AI agents and contributors

PhotoCraft is an open-source, native, Photoshop-comparable image editor written in **Rust only** (no JavaScript or TypeScript). The product name is always written **PhotoCraft** (`{Function}Craft` in PascalCase, like its siblings ArtCraft, ArtCraftX, DesignCraft, DrawCraft, EffectCraft, FilmCraft, LightCraft, PrintCraft) in user-facing text: UI, window titles, About, installers, release names, docs prose. Machine names stay lowercase: crates (`photocraft-*`), binaries, file names, ids (`ai.storyteller.photocraft`). Standards and learnings shared across the crafting apps live in `../craftrules` (read its `README.md`). Contribute reusable learnings there, never code; repos don't share code. The goal is 1:1 Photoshop parity (same menus, shortcuts, behaviour and file fidelity) with better performance, and every feature drivable by agents. Read this file first, then `docs/`.

## 1. Orientation (5 minutes)

| Read | Why |
|---|---|
| `docs/architecture.md` | Crate map, dependency layers, the engine/UI seam, document model |
| `docs/development.md` | Build, test, run, drive the app programmatically, debug tricks |
| `docs/contributing.md` | Rules: clean-room, tests, layering, style, commits; the "add a command" checklist |
| `docs/control-protocol.md` | JSON control channel: how agents drive and screenshot the running app |
| `docs/ui-design.md` | Design tokens, themes, widgets, and how to match Photoshop's look |
| `docs/roadmap.md` | Milestones and the **current focus** |
| `docs/parity.md` | Generated list of every Photoshop menu item, live or missing |
| `crates/<name>/README.md` (where present) | Public API of that crate |

## 2. Workspace map

```text
crates/
  geom cms color raster      L0 foundation (geometry, ICC colour management, pixel formats + blend math, COW tiles)
  psd codecs                 L0 standalone format crates (no workspace deps; publishable)
  doc                        L1 document model (layers, masks, adjustments, effects, smart objects: pure data)
  ops paint algo text vector L2 history, brush engine, imaging algorithms, type engine, paths/shapes
  compose gpu format         L3 CPU compositor (the oracle), wgpu compositor, .pcraft native format
  io                         L4 document <-> PSD / flat formats
  engine                     L5 Session + command registry (every action is a command)
  ui-egui automation         L6 egui shell (thin: all actions go through the engine); MCP server
  testkit                    test helpers
apps/
  photocraft                 desktop app (eframe/wgpu), TCP control server
  photocraft-cli             headless CLI (convert/info/run/batch/commands/mcp)
  photocraft-web             the same app in the browser (trunk + wasm-bindgen)
xtask/                       cargo xtask layers | wasm | ci | stats | corpus | parity
```

**Layering is enforced** by `cargo xtask layers`. A crate may depend only on lower layers. `psd`, `codecs` and `cms` depend on nothing in the workspace. Nothing below `ui-egui` may use egui, eframe, winit or rfd. A new crate must be registered in `xtask/src/layers.rs`.

## 3. Golden rules

1. **Everything is a command.** New user-visible behaviour = a command in the engine (`crates/engine/src/*_cmds.rs`, registered in `commands.rs`) with id, label, menu path, shortcut, params doc, `enabled` and `run`, plus tests. The UI, CLI, control channel and MCP all dispatch commands by id. Use the **exact id from `crates/ui-egui/src/menu_catalog.rs`** and the menu item goes live automatically. Only pure view/window state (zoom, panels, screen mode) belongs to the shell (`menus.rs` `UI_COMMANDS`).
2. **No format or colour assumptions.** Bit depth (8/16/32f) and colour model (RGB/Gray/CMYK/Lab…) are runtime data. Never introduce a `u8`-only pixel path in public APIs. Never assume sRGB: colour conversions go through `photocraft-cms` (`Transform`, `transform::cached`). Test at several depths.
3. **Clean-room.** We studied Photon Studio (proprietary) and Photoshop for *behaviour and look only*. Never copy their code, shaders, profiles or assets. Implement from public specs (Adobe PSD spec, ICC, ISO 32000 blend modes, papers) and observation. Third-party assets must be permissively licensed and recorded (see `assets/`, `docs/images/SOURCES.md`).
4. **Tests are the gate.** Every change comes with tests. Format crates use round-trip, synthetic-generator, oracle and fuzz tests. Keep the PSD corpus results and the parity floor (`crates/ui-egui/src/parity.rs`) from regressing.
5. **The UI is thin and data-driven.** UI state lives in `ui-egui/src/state.rs` (serde), so the control channel can read and drive it. Colours and radii come from `theme::Tokens`, never hard-coded.
6. **Verify UI changes visually.** Render offscreen with `cargo run -p photocraft-ui-egui --example snapshot` (no window, no focus stealing), or launch with `--control` and take `ui.screenshot`. Look at the PNG. Demo images must be public-domain art, never personal photos. When fetching assets, never put a person's name, email or other personal details in requests (User-Agent, headers, URLs); use a generic `Photocraft-dev` User-Agent.
7. **Never break wasm.** L0–L6 must `cargo check --target wasm32-unknown-unknown` (run `cargo xtask wasm`). File-system code is `cfg(not(target_arch = "wasm32"))` or goes through the platform services.
8. **Performance is a feature.** Benchmark heavy operations on a 24–36 MP image in release. Work per tile in parallel (rayon), skip empty tiles, never scan a full surface per frame (cache per revision), and record before/after timings in the dev log.

## 4. Picking work

Priorities: important infrastructure first, then low-hanging parity, then the long tail.

1. `docs/roadmap.md` → **Current focus**.
2. `cargo xtask parity` → `docs/parity.md` lists every missing menu item, grouped by menu. Low-hanging fruit is usually a missing command whose algorithm already exists in `algo`, `paint`, `vector` or `text`.
3. `log/devlog.md` → the "Still open" bullets of recent entries.

When parity rises, raise `FLOOR` in `crates/ui-egui/src/parity.rs` (never lower it).

## 5. Before you finish a task

```sh
cargo test -p <crates you touched>
cargo clippy -p <crates> --all-targets -- -D warnings
cargo xtask layers
cargo xtask wasm            # if you touched L0–L6
cargo xtask parity          # if you added commands; commit the regenerated docs/parity.md
```

Then append a terse entry to `log/devlog.md` (what landed, numbers, what's still open). Sessions can end abruptly (crashes, context limits), so the dev log plus a green tree is how the next agent picks up. Keep the tree building at every step.

## 6. Parallel agents

- Use your own target dir (`CARGO_TARGET_DIR=target/agent-<name>`) to avoid the Cargo build lock, and edit only the files you own. Shared files (`engine/src/lib.rs`, the `v.extend(...)` list in `engine/src/commands.rs`, `ui-egui/src/menus.rs`, `state.rs`) get small, surgical edits; re-read before editing.
- Put new commands in a **new module** (`engine/src/<area>_cmds.rs` with a `specs()` function) rather than growing a shared file.
- If someone else's in-progress edit breaks the build, wait and retry; don't fix their files.
- Keep every `Cargo.toml` valid at all times: the `crates/*` glob means one broken manifest breaks everyone's build. **Create or rewrite manifests atomically**: write to a temp file outside `crates/`, then `mv` it into place.
- Disk space: each target dir is about 10 GB. Delete `target/agent-*` dirs of finished agents.

## 7. Where things are tracked

- `docs/roadmap.md`: milestones M0–M12, status and the current focus.
- `docs/parity.md`: generated Photoshop menu coverage.
- `docs/releasing.md`: cutting a release (`cargo xtask version`, the `release` branch), signing secrets, packaging scripts in `packaging/`.
- `../craftrules/release/playbook.md`: how every storytold app builds signed release binaries (the canonical recipe; `docs/release-playbook.md` just points there); `docs/releasing.md` is PhotoCraft's specifics.
- `plan/` (local, gitignored): research, parity plan, execution plan, estimates.
- `log/` (local, gitignored): the dev log.

## 8. Keeping the native format complete

`photocraft-format` deliberately fails to compile when a `photocraft-doc` struct gains a field, so
nothing is silently dropped from `.pcraft` saves. When you add a doc field, add it to
`crates/format/src/manifest.rs` and `convert.rs` with `#[serde(default)]` so older files still load.
If the field has a PSD equivalent, map it in `crates/io` too, and keep unknown PSD blocks verbatim.

---
> Source: [storytold/photocraft](https://github.com/storytold/photocraft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
