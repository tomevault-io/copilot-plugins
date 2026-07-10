## fcemulator

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

`fc` is a from-scratch, cycle-driven NES/Famicom emulator **core** in Rust with four frontends, plus an MCP layer that exposes the emulator to LLM agents. See `README.md` and `docs/需求文档.md` (the authoritative spec) for background; `docs/路线图.md` and `docs/策划案-基于现状-可行性评估.md` for direction.

## Workspace layout

Cargo workspace members: **`fc-core`**, **`fc-cli`**, **`fc-mcp`**, **`fc-gui`**.
**`fc-tauri` is intentionally `exclude`d from the workspace** — it builds on its own toolchain (`npm` + Tauri). `cargo build` at the root does **not** touch it.

| Crate | Role |
|---|---|
| `fc-core` | Pure emulator logic — CPU/PPU/APU/mappers/bus/save-state/debugger/cheats. **No IO, no render, no audio deps.** All frontends drive it through `ControlDeck` (`control_deck.rs`). |
| `fc-cli` | The `fc` binary: headless run, ROM tests, disassembler, `mcp` server, `tauri-bridge`. |
| `fc-gui` | egui + wgpu desktop GUI, `cpal` audio. Paces emulation to the audio clock (`main.rs` runs frames while `audio.buffered() < ~50ms`). |
| `fc-mcp` | stdio JSON-RPC 2.0 MCP server (`emu_*` tools) wrapping `ControlDeck`. |
| `fc-tauri` | Tauri 2 + Vue 3 + Pinia + PixiJS player/IDE UI. Rust backend in `src-tauri/`, frontend in `src/`. |

## Build / run / test

```sh
# Workspace (everything EXCEPT fc-tauri)
cargo build                 # or --release
cargo test                  # unit tests live in fc-core (~5); other crates have none
cargo test -p fc-core <name># run a single test by name substring

# egui GUI
cargo run -p fc-gui --release -- roms/SuperMarioBro.nes

# CLI (`fc` binary) — subcommands: run | test | testsuite | disasm | info | dbg | mcp | tauri-bridge
target/debug/fc run roms/SuperMarioBro.nes --frames 600 --shot out.png --autostart
target/debug/fc test  nes-test-roms/.../nestest.nes --entry C000   # nestest: PASS == $0002==0000
target/debug/fc testsuite <blargg-rom...>                          # blargg $6000 protocol scorer
```

ROM for manual testing: **`roms/SuperMarioBro.nes`** (note: not repo root).

### fc-tauri (built/run separately — NOT via root `cargo`)

```sh
npm --prefix fc-tauri run tauri dev      # vite on :5183 + cargo-built app; watches src-tauri for Rust changes
( cd fc-tauri && npx vue-tsc --noEmit )  # frontend type-check (must run from fc-tauri/ for tsconfig)
cargo build --manifest-path fc-tauri/src-tauri/Cargo.toml   # backend-only compile check
```
Restart gotcha: `tauri dev` spawns `target/debug/fc-tauri` which **orphans** when the dev tree is killed — kill it by its bare name too, or you get a second window + duplicate audio/sockets.

## Core architecture (the parts that require reading several files)

**Lock-step clocking is the central invariant.** `cpu.rd/wr` (in `cpu.rs`) call `bus.tick()` (in `bus.rs`) **before** the memory access; each tick advances the PPU **3 dots** + APU **1 cycle** (+ services DMC DMA). So CPU/PPU/APU stay synchronized at sub-instruction granularity with **no per-game hacks**. PPU NMI edges propagate back through `bus.take_nmi()`/`poll_nmi()`. Changing read/access ordering changes timing behavior — be deliberate.

**`ControlDeck` is the single facade.** Every frontend (`fc-cli`, `fc-gui`, `fc-mcp`, `fc-tauri`) drives the machine only through `control_deck.rs`: `run_frame()` (returns `false` when halted at a breakpoint), `frame_buffer()`, `drain_audio()`, `save_state()/load_state()`, debugger (`add_breakpoint`/`set_breakpoint_enabled`/`step_instruction`/`is_halted`), and cheats. Add cross-frontend capability here, not in individual frontends.

**Mappers** (`mapper.rs`): enum dispatch over a `MapperOps` trait (NROM/MMC1/UNROM/CNROM/AxROM/MMC3/MMC2/MMC4/ColorDreams/GxROM/Codemasters). MMC3 scanline IRQ is driven by **A12 edges** filtered with the PPU's monotonic `master_cycle`; MMC2/4 CHR latch is notified **after** `ppu_read`.

**fc-tauri front/back split** (`src-tauri/src/emu.rs`): a worker thread runs `ControlDeck` paced by the **native cpal audio clock** (audio is the master — run frames while the output ring is below target; see `audio.rs`). `cpal::Stream` is `!Send`, so it lives entirely on the worker thread. Tauri commands return **raw binary** (`tauri::ipc::Response`, never JSON) for the ~240 KB/frame transfer; the frontend pulls the *latest* frame on `requestAnimationFrame` (old frames overwritten, never queued). Frontend state is in **Pinia** stores (`src/stores/`); UI follows the mockups in **`ui设计/<page>/image.png`** — read those, do not invent layouts. Pinia stores call `acceptHMRUpdate` at the bottom — required, or store-action edits silently don't hot-patch the running app.

## Accuracy testing protocol

- `fc test <rom> --entry <hex>`: sets PC, runs, reads result code at `$0002` (`0000` = pass). Used for nestest.
- `fc testsuite <roms...>`: blargg `$6000` protocol — waits for signature `DE B0 61` at `$6001..3`, then `$6000` status (`0x00` pass, `0x80+` running, else fail code). Scores PASS/FAIL/TIMEOUT.

## Dev-time AI / MCP (registered in `.mcp.json`, project scope)

- **`fc-ide`** (`fc ide-mcp`): semantic creative IDE API hosted inside the running Tauri process at `/tmp/fc-tauri-ide-mcp.sock`; tools create/open projects, write source/CHR/map/music resources, bind map↔CHR, build, run, and notify the visible UI through Tauri events.
- **`fc-emu`** (`fc emu-mcp`): live emulator MCP hosted inside the running Tauri process at `/tmp/fc-tauri-emu-mcp.sock`; it drives the same `EmuState` used by the visible player/IDE preview, including ROM load, frame stepping, input, memory, breakpoints, traces, event dumps, and `emu_capture_screen`.
- **`fc-emu-core`** (`fc mcp`): optional headless core MCP for pure emulator work without the Tauri window.
- **`fc-tauri`** (`fc tauri-bridge`): debug the **live** running Tauri window. `tauri_eval` runs JS in the real webview (read DOM, Pinia `window.__emu`/`window.__lib`, navigate) — works with no special permission. `tauri_screenshot` needs macOS **Screen Recording** permission. Plumbing: dev-only `tauri-plugin-mcp-gui` opens a Unix socket at `/tmp/fc-tauri-mcp.sock` (init in `lib.rs`, `#[cfg(debug_assertions)]`); the frontend `devtools.ts` answers the `execute-js` round-trip; `fc tauri-bridge` bridges MCP↔socket.

## Conventions

- Keep `fc-core` IO-free: file/audio/render/dialog code belongs in the frontends, never the core.
- Temporary `FC_TRACE`-gated `eprintln!` instrumentation for timing investigation currently lives in `fc-core/src/ppu.rs:182,248` (leftover VBL trace) — strip such probes before considering work done.

---
> Source: [Sundykin/fcemulator](https://github.com/Sundykin/fcemulator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-10 -->
