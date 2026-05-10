## videos

> **videOS** is a native macOS video player built entirely in **Swift + SwiftUI**, powered by **libVLC** for media playback. It is developed **without Xcode** — built, tested, and packaged entirely from the CLI using Swift Package Manager.

# videOS — Ultra-Advanced macOS Video Player

## Overview

**videOS** is a native macOS video player built entirely in **Swift + SwiftUI**, powered by **libVLC** for media playback. It is developed **without Xcode** — built, tested, and packaged entirely from the CLI using Swift Package Manager.

## Architecture

### Tech Stack
- **Language:** Swift 5.9+
- **UI Framework:** SwiftUI (macOS 13+)
- **Playback Engine:** libVLC 3.x via C interop
- **Persistence:** Codable + JSON (no CoreData/SwiftData dependency)
- **Build System:** Swift Package Manager + Make
- **Packaging:** Custom shell scripts → `.app` bundle

### libVLC Bridging Strategy

videOS bridges to libVLC's C API directly — no VLCKit, no Objective-C wrappers.

```
┌─────────────────────────────────────────────┐
│  SwiftUI Views                              │
│  ├── PlayerView (NSViewRepresentable)       │
│  ├── ControlBar                             │
│  └── LibraryView                            │
├─────────────────────────────────────────────┤
│  PlayerEngine (Swift class)                 │
│  ├── Wraps libvlc_media_player_t            │
│  ├── Publishes state via Combine            │
│  ├── C callback → Swift via pointer magic   │
│  └── Thread-safe: VLC callbacks on bg queue │
├─────────────────────────────────────────────┤
│  CLibVLC (SPM system library target)        │
│  ├── module.modulemap → libvlc headers      │
│  ├── shim.h → focused C type declarations   │
│  └── Links: -lvlc at build time             │
├─────────────────────────────────────────────┤
│  libvlc.dylib (from VLC.app or Homebrew)    │
└─────────────────────────────────────────────┘
```

**Key bridging details:**
- `CLibVLC/include/shim.h` declares opaque VLC types and the subset of functions we use
- `CLibVLC/module.modulemap` maps the shim to a Swift-importable module
- `PlayerEngine` uses `Unmanaged<PlayerEngine>` to pass `self` through C callbacks
- VLC events fire on background threads → dispatched to `@MainActor` via `DispatchQueue.main`
- NSView handed to VLC via `libvlc_media_player_set_nsobject()` for zero-copy rendering

### Module Structure

```
Sources/
├── videOS/                    # Main app target
│   ├── App.swift              # @main SwiftUI entry point
│   ├── AppDelegate.swift      # NSApplicationDelegate for menu/lifecycle
│   ├── Models/                # Pure data types (Codable)
│   ├── Services/              # Business logic, VLC integration
│   ├── ViewModels/            # Observable state for views
│   ├── Views/                 # SwiftUI views
│   │   └── Components/        # Reusable UI components
│   └── Utilities/             # Formatters, helpers
└── CLibVLC/                   # C bridging module for libVLC
    ├── include/shim.h         # libVLC type/function declarations
    └── module.modulemap       # Swift module map
```

## Build & Run

### Prerequisites

```bash
# Install VLC (provides libvlc.dylib)
brew install --cask vlc

# Or install libVLC headers + lib directly
brew install vlc
```

### Commands

```bash
make deps          # Install/verify dependencies
make build         # Debug build
make release       # Optimized release build
make run           # Build and launch
make package       # Create videOS.app bundle
make test          # Run tests
make clean         # Clean build artifacts
```

### How Linking Works

The build finds libVLC in these locations (in order):
1. `/Applications/VLC.app/Contents/MacOS/lib/`
2. `/usr/local/lib/` (Homebrew Intel)
3. `/opt/homebrew/lib/` (Homebrew Apple Silicon)
4. Custom `VLC_LIB_PATH` environment variable

## UI/UX — Glass Morphism Design

### Design Language
- **Glass panels** using SwiftUI `.ultraThinMaterial` / `.regularMaterial`
- **Vibrancy** text and icons on translucent surfaces
- **Dark-first** with automatic light mode support
- **Floating controls** that auto-hide during playback
- **Smooth animations** on all state transitions (spring-based)

### Layout

```
┌──────────────────────────────────────────────────┐
│ ◉ ◉ ◉  videOS            🔍  ☰               │ ← Title bar (transparent)
├────────┬─────────────────────────────────────────┤
│        │                                         │
│  📁    │                                         │
│ Library│         VIDEO CANVAS                    │
│        │       (NSView → VLC)                    │
│  🎵    │                                         │
│Playlists│                                        │
│        │                                         │
│  🌐    │                                         │
│Streams │                                         │
│        ├─────────────────────────────────────────┤
│  🔖    │   advancement ◀◀  ▶  ▶▶  🔊 ─── │ ← Control bar (glass)
│Bookmarks│  00:12:34 / 01:42:00  [A] [S]        │
├────────┴─────────────────────────────────────────┤
└──────────────────────────────────────────────────┘
```

### Key Interactions
- **Double-click** video → fullscreen toggle
- **Scroll** on video → volume
- **Scroll** on seek bar → scrub with thumbnail preview
- **Space** → play/pause
- **← →** → seek ±10s (shift: ±30s, alt: ±5s)
- **⌘F** → fullscreen
- **⌘O** → open file
- **⌘U** → open URL stream
- **⌘I** → media info overlay
- **⌘,** → settings

## Features (Phase 1 — Core)

- [x] Project scaffold with SPM + libVLC bridging
- [ ] Video playback (local files)
- [ ] Play/pause, seek, volume controls
- [ ] Fullscreen mode
- [ ] Subtitle track selection
- [ ] Audio track selection
- [ ] External subtitle file loading (.srt, .ass, .ssa)
- [ ] Media library (scan folders, remember files)
- [ ] Playlists (create, edit, reorder, delete)
- [ ] Bookmarks within a video
- [ ] Resume playback from last position
- [ ] Network stream playback (HTTP, RTSP, HLS)
- [ ] Keyboard shortcuts (VLC-like defaults)
- [ ] Glass morphism UI with auto-hiding controls

## Features (Phase 2 — Advanced)

- [ ] Equalizer (audio)
- [ ] Playback speed control (0.25x–4x)
- [ ] A-B loop
- [ ] Screenshot / snapshot
- [ ] Picture-in-Picture
- [ ] Chapter navigation
- [ ] Media metadata display (codec, bitrate, resolution)
- [ ] Drag-and-drop file/folder import
- [ ] Recently played list
- [ ] Multiple windows

## V2 Roadmap — Legendary Features

### AI Subtitle Tools
- **Auto-generate subtitles** using Whisper (local or API)
- **Translate subtitles** on-the-fly between languages
- **Subtitle search** — find the moment someone says a word
- **Smart sync** — auto-align mismatched subtitle timing

### Clip Export
- **Trim & export clips** from any video
- **GIF export** with subtitle burn-in
- **Batch export** multiple bookmarked segments
- **Re-encode** with preset profiles (web, mobile, archive)

### Metadata Enrichment
- **Auto-fetch** movie/TV metadata from TMDb/OMDb
- **Poster art** and backdrop integration
- **Smart rename** files based on matched metadata
- **NFO/XML** sidecar generation for Plex/Kodi/Jellyfin

### Advanced Playback
- **HDR tone mapping** for SDR displays
- **Audio normalization** across playlist
- **Crossfade** between playlist items
- **Watch party** — synchronized playback over LAN
- **DLNA/UPnP** casting to smart TVs

### Library Intelligence
- **Duplicate detection** by content hash
- **Auto-organize** into Movies/TV/Music Videos
- **Smart playlists** with rule-based filters
- **Playback statistics** and history analytics

## Code Conventions

- **No Xcode project files** — SPM only
- **SwiftUI-first** — AppKit only where required (NSView for VLC, menus)
- **MVVM** — Views observe ViewModels, ViewModels use Services
- **Combine** for reactive state propagation
- **@MainActor** on all UI-bound types
- **Explicit error handling** — no force unwraps in production code
- **JSON persistence** in `~/Library/Application Support/videOS/`
- **Modular services** — each concern in its own file

## Persistence Layout

```
~/Library/Application Support/videOS/
├── library.json          # Scanned media items
├── playlists.json        # User playlists
├── bookmarks.json        # Per-media bookmarks
├── playback-state.json   # Resume positions
└── settings.json         # User preferences
```

## Testing

```bash
swift test                              # All tests
swift test --filter PlayerEngineTests   # Specific suite
```

Tests mock the VLC layer via protocol (`MediaPlayerProtocol`) to enable unit testing without libVLC installed.

---
> Source: [Worth-Doing/videOS](https://github.com/Worth-Doing/videOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
