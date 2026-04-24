## termwave

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is termwave

Terminal audio visualizer for macOS. Renders real-time spectrum bars, waveforms, and oscilloscopes from mic input or system audio. Built with Rust (cpal + ratatui) and a companion Swift binary (`termwave-tap`) for system audio capture via ScreenCaptureKit.

## Build & Development Commands

Uses `just` as the task runner:

- `just build` — debug build (Rust + Swift)
- `just release` — release build
- `just install` — build release and install both binaries to `~/.cargo/bin`
- `just run [ARGS]` — debug run (e.g. `just run --mode wave --theme fire`)
- `just run-release [ARGS]` — release run
- `just lint` — `cargo clippy -- -D warnings`
- `just fmt` — `cargo fmt`
- `just fmt-check` — check formatting without modifying
- `just test` — `cargo test`
- `just clean` — clean both Rust and Swift build artifacts

Run a single test: `cargo test test_name`

The Swift tap binary must be built separately (`cd tap && swift build`) or via `just build`/`just install`.

## Architecture

Two-process design: the Rust binary handles all visualization, the Swift binary (`tap/`) only captures system audio.

**Main loop** (`main.rs`): CLI parsing (clap) → load config → start audio capture → render loop at target FPS. The render loop is: read samples from shared buffer → FFT → bin/smooth → draw.

**Data flow**: Audio thread writes interleaved samples into `Arc<Mutex<Vec<f32>>>` ring buffers (mono + stereo pair). Render thread reads from these buffers each frame.

**Key modules**:
- `audio.rs` — Two capture paths: `start_capture` (cpal device input) and `start_tap` (spawns `termwave-tap` subprocess, reads raw f32 from its stdout). `CaptureHandle` enum keeps the stream/child alive via RAII.
- `analysis.rs` — Dual-resolution FFT (4096 for bass <200Hz, 2048 zero-padded to 4096 for mids/highs), logarithmic frequency binning with equalizer curve, frame smoothing, monstercat envelope smoothing, noise gate, `Gravity` (frame-rate independent bar fall-off), and `AutoSensitivity` (asymmetric gain: fast attack, slow multiplicative recovery).
- `render.rs` — All ratatui terminal UI. Visualizer functions (`render_spectrum`, `render_wave`, `render_scope`, `render_stereo`) take a `&mut Frame` so they can be composed in a single `terminal.draw` call. Settings is a non-blocking overlay (`SettingsState` + `render_settings`) rendered on top of the visualizer. Device menu and help screen are still blocking. All UI chrome (borders, titles, labels) uses theme colors.
- `theme.rs` — Loads theme definitions from TOML files in `~/.config/termwave/themes/`. Each theme file has a `[colors]` table of named hex colors and a `[visualizer]` table that references those names for gradient stops, wave_color, and scope_color. No theme data is hardcoded in Rust.
- `config.rs` — Persists settings to `~/.config/termwave/config.toml` via serde/toml. Settings changed at runtime (theme, smoothing, etc.) are saved automatically.

**System audio** (`tap/Sources/main.swift`): Uses ScreenCaptureKit to capture system audio output, mixes to mono f32, writes raw bytes to stdout. Requires macOS 13+ and Screen Recording permission.

**Config persistence**: Runtime settings auto-save to `~/.config/termwave/config.toml`. CLI args override saved config. Defaults: 60fps, 50–10000Hz range, 0.3 smoothing, monstercat on.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jrf) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
