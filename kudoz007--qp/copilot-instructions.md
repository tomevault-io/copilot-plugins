## qp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Verify

Use Xcode MCP tools when available (preferred over generic shell commands):

- **Build**: `BuildProject` MCP tool
- **Check diagnostics after edits**: `XcodeRefreshCodeIssuesInFile` (fast, no full build needed)
- **Inspect build errors**: `GetBuildLog` MCP tool
- **Read/write project files**: Use `XcodeRead` / `XcodeUpdate` / `XcodeWrite` (paths are workspace-relative, e.g. `QuPi/QuPi/Foo.swift`). Note: `XcodeUpdate` does not always flush to disk for pre-existing files — verify with a filesystem `Read` after editing, and fall back to the filesystem `Edit` tool if needed.

There is no test target and no linter configuration in this project.

## Architecture

QuPi is a **macOS-only menu bar app**. The entry point (`ContentView.swift`) declares:
- A `MenuBarExtra` (dropdown UI via `MenuBarContentView`)
- Two `WindowGroup` scenes keyed on `MediaItem`: `"video-player"` (780×460) and `"music-player"` (340×660)
- A `Settings` scene hosting `SettingsView`

### Central state — `AppState`

`AppState` is a `@MainActor @Observable final class` passed as an `@Environment` to all views. It owns:
- The catalog (`itemsBySection`, `drillPath`, `childrenByItemID`)
- Search state and debounced deep-search task
- Playback reporting (fans out to server timeline APIs and scrobblers)
- Auto-continue logic (next movie / next episode / next track)

All preference reads go through `UserDefaults` via `SettingsKeys` (non-secret) or `KeychainStore` via `KeychainKeys` (tokens/secrets). There is no SwiftData or CoreData.

### MediaProvider protocol

`MediaProvider` (defined in `MediaModels.swift`) abstracts where content comes from. `AppState.providers` assembles an ordered list each time it's called:

1. One `PlexMediaProvider` per configured server
2. `JellyfinMediaProvider` (if configured)
3. `LocalMediaProvider` — serves locally downloaded files; only registered when it `hasContent`
4. `SampleMediaProvider` — fallback when no server and no local content

Adding a new backend means conforming to `MediaProvider` and inserting the provider in `AppState.providers`.

**De-duplication**: local items are filtered against server item IDs so a downloaded file doesn't appear twice when the originating server is connected.

### Media hierarchy

```
MediaType (movies / tvShows / music)
  └─ MediaKind (movie | show → season → episode | artist → album → track | playlist)
```

`MediaKind.isExpandable` determines whether tapping opens a child carousel (drill-down) or launches the player. The drill path per section lives in `AppState.drillPath`.

### Settings structure

`SettingsView` is a `TabView` with six tabs: General, Accounts, Libraries, Visuals, Playback, Data. Each tab is its own `View` struct in its own file.

- **Accounts**: Plex (multi-server via `PlexServerStore` + Keychain) and Jellyfin sign-in, plus Trakt and Last.fm scrobbler credentials.
- **Libraries**: Plex/Jellyfin library multi-select (stored as comma-joined strings in `SettingsKeys`), plus a Local Library section showing per-type folder and usage.
- **Data → Downloads**: Per-type download folder (security-scoped bookmark via `DownloadManager.setFolder(_:for:)`), storage cap, and download-level toggles (`DownloadLevel` enum maps to `MediaKind`).

### Downloads & local playback

`DownloadManager` (singleton `shared`) manages:
- Security-scoped folder bookmarks stored in `UserDefaults` (sandbox-compatible)
- A JSON index per type tracking downloaded items and their relative paths
- Storage usage and the `localURL(for:)` lookup used by `AppState.streamURL`

`LocalMediaProvider` reads the index and also scans for unindexed files dropped manually into the folder.

### Playback

`PlayerView` hosts either `AVPlayer` (for AVFoundation-compatible containers) or `VLCPlayerBridge` (libVLC, for MKV/AVI/etc.). `isAVFoundationPlayable(_:)` in `MediaModels.swift` decides which engine to use based on file extension; network URLs always use AVFoundation.

Playback position is persisted by `PlaybackProgressStore` (UserDefaults-backed) and drives the **Continue…** section. The `ContinueMusicGrouping` preference collapses in-progress tracks into their parent album/playlist cell.

### Scrobbling

`ScrobbleClients.swift` implements Trakt (movies) and Last.fm (music) reporting. Called from `AppState.scrobble(item:state:progressPercent:)` on playback state transitions (not on periodic `.playing` ticks).

### Key naming / style notes

- All `@Observable` classes must be `@MainActor`.
- Settings keys are centralised in `SettingsKeys` (UserDefaults) and `KeychainKeys` (Keychain) — add new keys there, not as inline string literals.
- The app name is **QuPi**. Avoid "QuickPlex" in user-facing strings and comments.

---
> Source: [KuDoZ007/QP](https://github.com/KuDoZ007/QP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
