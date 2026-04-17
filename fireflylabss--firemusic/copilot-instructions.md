## firemusic

> **firemusic** is a high-performance, minimalist CLI music player built with Rust. It leverages the **MPV** engine via `libmpv2` to provide a sleek and robust playback experience directly in the terminal.

# 💎 Project Context: firemusic (fire music)

## Project Overview
**firemusic** is a high-performance, minimalist CLI music player built with Rust. It leverages the **MPV** engine via `libmpv2` to provide a sleek and robust playback experience directly in the terminal.

- **Purpose:** Provide a tactical, zero-leak CLI interface for playing local files and web streams, with advanced discovery and downloading capabilities.
- **Main Technologies:** Rust, `libmpv2` (MPV 2.0+), `yt-dlp` (for discovery, metadata, and downloading), `crossterm` (tactical UI).

## Key Features
- **Zero-Leak UI Engine:** Strictly managed 3-line fixed UI block. Uses absolute cursor positioning and atomic updates.
- **Interactive Discovery Hub:** Search music across platforms (YouTube, YouTube Music, SoundCloud) directly from the CLI via `msc --search`.
- **Advanced Download System:** Multi-format interactive wizard with metadata/thumbnail embedding and subtitle selection.
- **Audio FX System:** Real-time manipulation of **Pitch** (frecuency) and **Equalizer** presets (Bass+, Treble+, Rock, etc.).
- **Smart Navigation:** Automatic pagination for search results and a post-track decision menu (repeat, new search, back to results).
- **Format-Aware Subtitles:** Intelligent validation to only offer subtitle embedding for compatible containers (MP4, MKV, M4A).

## Building and Running

### Prerequisites
- **libmpv 2.0+** development files must be installed.
- **yt-dlp** must be in the system PATH.

### Key Commands
- **Manual Install:** `cargo install --path .` (installs both `msc` and `firemusic` binaries).
- **Discovery Hub:** `msc -s` or `msc -s "query"`.
- **Download Wizard:** `msc --download`.
- **Direct Play:** `msc <path-or-url>`.

## Development Conventions

### UI Rendering (The "Zero-Leak" Engine)
- The UI must always be confined to a **strictly managed 3-line block**.
- Use **relative cursor positioning** and `ClearType::FromCursorDown` for menu transitions to prevent ghosting.
- Status indicators use minimalist glpyhs: `▶` (playing), `⏸` (paused).
- The progress bar uses a `█` playhead in `dark_red`.

### Tactical Selection (`tactical_select`)
- Custom-built selection engine for search and menus.
- **Enter** selects and confirms immediately (fast flow).
- **Space** toggles multi-select without moving the cursor.
- Automatic pagination for lists > 15 items with `←/→` navigation.

### Audio Control
- **Pitch:** Controlled via `,` and `.` (0.05x increments).
- **EQ:** Cycled via `e` using native MPV hardware filters (`bass`, `treble`).
- **Reset:** `0` clears all FX (speed, pitch, eq).

## Legal
- **License:** Apache License 2.0 (Copyright © 2026-Present Firefly Labs).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fireflylabss) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
