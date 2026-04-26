## rekordboxexplorermac

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RekordboxExplorerMac is a SwiftUI application for browsing Rekordbox USB libraries without opening the full Rekordbox application. It parses the binary `export.pdb` file format that Rekordbox uses to store library data on USB drives.

**Platforms:** macOS and iOS (separate targets, shared core code)

## Architecture

```
RekordboxExplorerMac/
├── Shared/                    # Cross-platform code (iOS + macOS)
│   ├── Core/
│   │   ├── PDBParser.swift    # Binary parser for Rekordbox export.pdb format
│   │   ├── BookmarkStore.swift # Security-scoped bookmark persistence
│   │   └── AppPreferences.swift # Shared user preferences
│   ├── Models/
│   │   ├── Track.swift        # Track data model
│   │   ├── Playlist.swift     # Playlist/folder hierarchy model
│   │   └── RekordboxDatabase.swift # Combined database container
│   └── Services/
│       ├── PDFExportService.swift  # PDF generation for track lists
│       └── SectionBuilder.swift    # Builds sections for PDF export
├── RekordboxExplorerMac/      # macOS-specific code
│   ├── Views/
│   │   ├── MainView.swift     # Main navigation split view
│   │   └── MacSettingsView.swift # macOS settings sheet
│   └── Services/
│       ├── RekordboxScanner.swift  # Finds export.pdb on USB
│       ├── FolderAccessService.swift # Folder picker/access
│       ├── ExportService.swift     # CSV/JSON export
│       └── Formatters.swift        # Duration/date formatting
└── RekordboxExploreriOS/      # iOS-specific code
    ├── Views/
    │   ├── ContentView.swift       # iOS root view
    │   ├── LibraryRootView.swift   # Library navigation
    │   ├── PlaylistView.swift      # Playlist browser
    │   ├── PlaylistTracksView.swift # Tracks in playlist
    │   ├── AllTracksView.swift     # All tracks list
    │   ├── TrackDetailView.swift   # Track detail sheet
    │   ├── TrackRowView.swift      # Track list row
    │   ├── SettingsView.swift      # iOS settings
    │   └── PDFExportHelpers.swift  # iOS PDF export UI
    └── Services/
        ├── IOSLibraryLoader.swift  # iOS library loading
        ├── DocumentPicker.swift    # File picker wrapper
        └── ShareSheet.swift        # UIActivityViewController wrapper
```

## Key Technical Details

### PDB Parser (`Shared/Core/PDBParser.swift`)
- Parses Rekordbox's binary `export.pdb` format
- Multi-pass parsing: lookups (artists, albums, genres, keys) first, then playlists, then tracks
- Uses a custom `DataView` struct for little-endian binary reading
- Page-based structure with row groups and presence flags

### Data Models
- **Track**: id, title, artist, album, genre, duration, bpm, key, rating, bitrate, filePath, dateAdded
- **Playlist**: id, name, parentId, isFolder, children, trackIds (hierarchical tree structure)
- **RekordboxDatabase**: Contains arrays of tracks and root playlists

### Platform Differences
- macOS uses `NavigationSplitView` with `Table` for track display
- iOS uses `NavigationStack` with `List` and detail sheets
- Both share `PDBParser`, models, and `PDFExportService`
- PDF rendering uses `UIGraphicsPDFRenderer` on iOS, `CGContext` on macOS

## Build Commands

```bash
# Build macOS app
xcodebuild -scheme RekordboxExplorerMac -destination 'platform=macOS' build

# Build iOS app
xcodebuild -scheme RekordboxExploreriOS -destination 'platform=iOS Simulator,name=iPhone 16' build

# Build both (from Xcode)
# Open RekordboxExplorerMac.xcodeproj and select appropriate scheme
```

## Common Development Tasks

### Adding a new track field
1. Add property to `Track` struct in `Shared/Models/Track.swift`
2. Parse it in `parseTrackRow()` in `PDBParser.swift`
3. Add UI column in `MainView.swift` (macOS) and `TrackRowView.swift`/`TrackDetailView.swift` (iOS)

### Adding export format
1. Add format case to `ExportService` (macOS) or create equivalent for iOS
2. Add menu item in toolbar

### Modifying PDF export
- Edit `PDFExportService.swift` in `Shared/Services/`
- Note: iOS and macOS use different rendering APIs but share layout logic

## File Access Notes
- App is sandboxed; uses `startAccessingSecurityScopedResource()` for folder access
- `BookmarkStore` persists access between launches using security-scoped bookmarks
- USB drives are accessed via folder picker, not direct path

## Preferences
- Stored via `@AppStorage` in `AppPreferences.swift`
- Shared keys: `colorSchemePreference`, `fontSizeMultiplier`
- Both platforms read/write same UserDefaults keys

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LubaKaper) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
