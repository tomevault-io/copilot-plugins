## kdeconnect-sms-tui

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Terminal UI for sending/receiving SMS/MMS via KDE Connect. Rust async app using ratatui for rendering and zbus for D-Bus communication with kdeconnectd.

## Build & Test Commands

```bash
cargo build                         # Debug build
cargo build --release               # Release build
cargo build --no-default-features   # Build without HEIF support
cargo test                          # Run all tests
cargo test --lib                    # Library tests only
cargo test <test_name>              # Single test
cargo clippy                        # Lint
```

A Nix flake is available (`nix develop`) with cargo-watch, cargo-nextest, and required native deps (libheif, dbus).

## Architecture

**Event-driven TUI** with three concurrent streams merged via `tokio::select!`:
- Terminal input events (crossterm, 250ms tick)
- D-Bus signals from kdeconnectd (message created/updated/removed, attachment received)
- Tick-based housekeeping (loading timeouts, connection retries)

### Key modules

- **`app.rs`** — Central `App` struct. Owns all state, runs the main event loop (`run_inner`), dispatches events by `Focus` enum. This is the largest file (~4600 lines).
- **`events.rs`** — `AppEvent` enum and event loop spawning (terminal + D-Bus signal listeners).
- **`state.rs`** — Persistent state (group names, archived/spam lists) saved to `~/.local/state/kdeconnect-sms-tui/state.toml`.
- **`contacts.rs`** — vCard parsing from `~/.local/share/kpeoplevcard/`, phone number normalization with suffix matching fallback.
- **`dbus/`** — D-Bus clients: `DaemonClient` (device discovery), `ConversationsClient` (SMS/MMS API), signal listeners.
- **`models/`** — Data structs: `Message`, `Conversation`, `Device`, `Attachment`. Messages parsed from D-Bus variant types.
- **`ui/`** — Rendering layer. `draw()` in `mod.rs` orchestrates layout. Each panel is a separate module (conversation_list, message_view, compose, popups, file_picker_popup). `theme.rs` for colors/styles.

### Key patterns

- **Focus enum** routes keyboard input to the correct panel handler.
- **Lazy message loading** — conversations track `messages_requested` vs `total_messages`; pages fetched on scroll.
- **Send cooldown** (2.5s) — suppresses daemon polling after send to prevent duplicate delivery.
- **Draft persistence** — `HashMap<i64, (String, usize)>` saves draft text + cursor per conversation thread.
- **Image state machine** — per-attachment `ImageState::Downloading | Loaded | Failed`, keyed by unique_identifier.
- **Emoji sanitization** — ZWJ sequences and variation selectors stripped before width calculation to prevent rendering misalignment.
- **`Arc<AtomicU64>` phone_request_epoch** — prevents race conditions in background message loading.

### D-Bus interface

Communicates with `org.kde.kdeconnect` on the session bus. Key operations: `activeConversations()`, `requestConversation(threadID, start, end)`, `replyToConversation(threadID, message, attachmentUrls)`, `requestAttachmentFile(partID, uniqueIdentifier)`.

## Runtime Requirements

- `kdeconnectd` running with a paired Android device
- D-Bus session bus
- Optional: `libheif` for HEIC/HEIF inline images
- Optional: `wl-copy`/`xclip`/`xsel` for clipboard support

---
> Source: [firecat53/kdeconnect-sms-tui](https://github.com/firecat53/kdeconnect-sms-tui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
