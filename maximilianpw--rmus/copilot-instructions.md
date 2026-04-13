## rmus

> rmus is a terminal-based music player inspired by cmus, built with Rust and Ratatui. It supports local file browsing and streaming from Qobuz and Tidal. MPV is used as the audio backend via IPC.

# CLAUDE.md

## Project Overview

rmus is a terminal-based music player inspired by cmus, built with Rust and Ratatui. It supports local file browsing and streaming from Qobuz and Tidal. MPV is used as the audio backend via IPC.

## Build & Development Commands

```bash
cargo build                                       # Debug build
cargo build --release                             # Release build (LTO, codegen-units=1, size-optimized, stripped)
cargo test --locked --all-features --all-targets  # Full test suite (CI command)
cargo test <test_name>                            # Run a single test
cargo fmt                                         # Format code
cargo fmt -- --check                              # Check formatting (CI command)
cargo clippy                                      # Lint
cargo doc --no-deps --all-features                # Build docs (CI runs this with nightly)
```

Nix flake provides the dev environment with Rust nightly toolchain and MPV. Use `nix develop` or direnv.

Runtime dependency: `mpv` must be available on PATH.

CI (`.github/workflows/ci.yml`): fmt, clippy, doc (nightly), test (macOS + Windows only).

## Architecture

The app follows a classic TUI event loop pattern: poll input → resolve keymap → execute action → render.

### Event Flow

1. **Crossterm events** are polled with 50ms timeout in `event.rs::handle_crossterm_events` (called from `app.rs::App::run`)
2. **Keymap resolution** (`keymap.rs::resolve_key`) maps KeyEvent → `KeyAction` enum (`Execute(Action)`, `DelegateToPanel`, or `None`), distinguishing global keys (quit, tab, settings, search) from panel-delegated keys
3. **Action execution** (`app.rs::App::execute`) updates state, controls the player, or manages panels
4. **Tick** (`app.rs::App::tick`) handles deferred async work: auth polling, config sync, pending searches

### Key Abstractions (Traits)

- **`MusicPlayer`** (`players/mod.rs`) — playback control (play, pause, seek, volume, next/prev). Returns `PlayerResult<T>`. `SafePlayer` wraps `MpvPlayer` with path validation and secure socket placement (0700 permissions on socket dir).
- **`MusicSource`** (`sources/mod.rs`) — provides albums and songs. `LocalFiles` (`sources/local.rs`) scans configured directories. `StreamingTab` is a placeholder source for streaming tabs (returns no albums; streaming uses search instead).
- **`StreamingService`** (`sources/streaming.rs`) — authentication, search, and stream URL resolution. Returns `Result<_, Box<dyn Error>>`. Implemented by `QobuzSource` and `TidalSource`.
- **`AppPanel`** (`ui/mod.rs`) — rendering contract for some panels (`LogPanel`, `RightPanel`, `SettingsPanel`). `LeftPanel` and `CenterPanel` have compatible render methods but don't implement this trait.

### Module Layout

- `app.rs` — App struct (main state machine), event loop, action dispatch
- `action.rs` — Action enum (all user-triggerable actions)
- `event.rs` — crossterm event polling (`handle_crossterm_events`, `POLL_TIMEOUT`)
- `keymap.rs` — keybinding resolution logic (`KeyAction` enum)
- `config.rs` — TOML config loading/saving via `directories::ProjectDirs` (macOS: `~/Library/Application Support/com.maximilianpw.rmus/config.toml`, Linux: `~/.config/com.maximilianpw.rmus/config.toml`)
- `players/mpv.rs` — MPV IPC implementation (JSON-RPC over Unix socket). Implements `Drop` to kill mpv process and remove socket. Playlist managed in Rust, not via MPV's built-in playlist. `previous()` seeks to 0 if >3s into track (cmus-style).
- `sources/local.rs` — `LocalFiles` implementation
- `sources/song.rs` — `Song` struct
- `sources/qobuz.rs` — Qobuz API client (MD5 password auth)
- `sources/tidal.rs` — Tidal API client (OAuth2 device code flow with polling)
- `ui/` — LeftPanel (album browser), CenterPanel (songs/search), RightPanel (now-playing), LogPanel
- `ui/input_line.rs` — reusable text input component (used by search and settings)
- `ui/widget.rs` — shared widget helpers (`now_playing_widget`, `list_from_strings`, border styling)
- `ui/settings/` — settings submodule: `SettingsPanel` (overlay), `AccountSettings`, `SourceSettings`

### Async Pattern

Streaming services use tokio internally but expose sync trait methods (blocking at the trait boundary). Long-running auth flows (Tidal device code) use a poll model: `authenticate()` returns `PendingUserAction`, then `poll_auth()` is called each tick until complete. Searches can be deferred if auth is still pending.

### Logging

Channel-based: `LogPanel::new()` returns `(LogPanel, Logger)`. `Logger` is cloneable and distributed to panels. Messages are drained each frame via `log_panel.poll()`.

### Error Handling

- Top level: `color_eyre::Result<()>`
- Player: `PlayerResult<T> = Result<T, PlayerError>` (structured errors)
- Streaming: `Result<_, Box<dyn std::error::Error>>` (dynamic errors)

### Testing

Tests are in `tests/e2e.rs` and `src/config.rs`. E2E tests use `App::new_for_test()` with a `MockStreamingService` and Ratatui's `TestBackend` for headless rendering verification. No MPV process is needed for tests.

### Known Issues

- `lofty` and `rpassword` are unused dependencies in `Cargo.toml`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/maximilianpw)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/maximilianpw)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
