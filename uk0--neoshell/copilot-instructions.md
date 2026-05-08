## neoshell

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

NeoShell — cross-platform SSH/server management tool (FinalShell alternative). Pure Rust, native GUI. Targets macOS, Windows, Linux.

## Tech Stack

- **GUI**: iced 0.13 (wgpu GPU-accelerated, native Rust)
- **Terminal**: VTE parser + custom canvas renderer
- **SSH**: Rust `ssh2` crate (libssh2 binding)
- **Encryption**: AES-256-GCM + Argon2id key derivation
- **Async**: tokio
- **State**: parking_lot RwLock/Mutex

## Build Commands

**Critical**: `/opt/bin/cc` is broken. All cargo commands require:
```bash
RUSTFLAGS="-C linker=/usr/bin/cc" CC=/usr/bin/cc CXX=/usr/bin/c++
```

```bash
cargo check           # Type check
cargo build           # Debug build
cargo run             # Run app
cargo test            # Run tests (terminal has 11 unit tests)
cargo clippy          # Lint
cargo build --release # Release build
```

`libssh2` required: `brew install libssh2` (macOS).

## Architecture

Cargo workspace with thin launcher + dynamic core library:

```
Cargo.toml              # Workspace root
launcher/
  src/main.rs           # Thin binary: dlopen core, restart loop, update swap
core/
  build.rs              # Windows resource compilation
  src/
    lib.rs              # Exports neoshell_run() + neoshell_version() via C ABI
    app.rs              # iced Application: state, update, view, subscriptions
    crypto/mod.rs       # AES-256-GCM encryption, Argon2id key derivation
    storage/mod.rs      # Encrypted connection vault (vault.json)
    ssh/mod.rs          # SSH session manager (ssh2 + background threads)
    terminal/mod.rs     # VTE terminal emulator (grid + parser)
    sshconfig.rs        # ~/.ssh/config parser
    ui/
      mod.rs            # UI module re-exports
      theme.rs          # Color constants
    updater.rs          # Background update checker/downloader
```

Build produces: `neoshell` (launcher binary) + `libneoshell_core.dylib` (cdylib with all logic).

### Data Flow
1. User interacts with iced GUI → generates `Message`
2. `update()` handles messages, dispatches `Task::perform` for async ops
3. SSH data arrives via `mpsc::Receiver<SshEvent>`, polled at 50ms intervals
4. Terminal grid updated with VTE parser, rendered on iced Canvas

### Key Patterns
- App uses iced functional API: `iced::application(title, update, view)`
- State machine: Setup → Locked → Main screen
- SSH sessions run on std::thread (ssh2 is blocking), communicate via channels
- Terminal: `TerminalGrid` implements `vte::Perform` for escape sequence handling
- Encrypted vault at `~/Library/Application Support/neoshell/vault.json`
- Two-layer encryption: master password → KEK (Argon2id) → DEK → connection data

### Keyboard Handling
- `event::listen_with` captures keyboard events when terminal is active
- `key_to_terminal_bytes()` converts iced key events to terminal escape sequences
- Keys NOT captured by widgets (text_input) are forwarded to active SSH session

## Core Features

1. SSH terminal with multi-tab sessions (implemented)
2. Connection manager with AES-256-GCM encrypted storage (implemented)
3. Master password vault with Argon2id key derivation (implemented)
4. VTE terminal emulator with 256-color + truecolor support (implemented)
5. SFTP file browser/transfer (planned)
6. Server monitoring (planned)

## Release & CI

- Tag-based releases trigger CI/CD builds (not main branch pushes)
- `dev` branch is local-only, never push to remote
- Two-artifact output: launcher binary + core dylib, no web runtime dependencies
- Updater checks https://neoshell.wwwneo.com/updates/update.json for new core library versions

---
> Source: [uk0/NeoShell](https://github.com/uk0/NeoShell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
