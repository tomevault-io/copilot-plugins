## diy

> This file provides guidance to Claude Code when working with this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Build Commands

```bash
# Build release version
./BuildRelease.sh

# Or directly with xcodebuild
xcodebuild -project DiskInventoryY.xcodeproj -scheme DiskInventoryY -configuration Release

# Build debug version
xcodebuild -project DiskInventoryY.xcodeproj -scheme DiskInventoryY -configuration Debug

# Create DMG for release
hdiutil create -volname "Disk Inventory Y" \
  -srcfolder "build/Build/Products/Release/Disk Inventory Y.app" \
  -ov -format UDZO "DiskInventoryY-1.0.dmg"
```

No test suite exists.

## Architecture Overview

Disk Inventory Y is a macOS SwiftUI application that visualizes disk space usage using treemaps. Pure Swift codebase with async/await concurrency. Rewrite of Disk Inventory X.

### Core Components

- **App/DiskInventoryYApp.swift** - App entry point, single-window application
- **App/AppState.swift** - Observable state management, orchestrates scanning, statistics calculation

### Models

- **Models/FileNode.swift** - Tree structure representing scanned filesystem. Uses `ObjectIdentifier` for identity (not UUID), `UInt16` kindId for memory efficiency. Stores path, size, children.
- **Models/FileKindRegistry.swift** - Singleton mapping file extensions to UInt16 IDs. UTType lookup happens once per unique extension, not per file. Reduces memory ~50 bytes/node.

### Services

- **Services/FileScanner.swift** - Actor-based async filesystem scanner. Uses `withThrowingTaskGroup` for parallel directory traversal (batch size 8). Scans all files including hidden. Supports cancellation.

### Views

- **Views/ContentView.swift** - Main view composing sidebar and treemap
- **Views/TreeMap/TreeMapView.swift** - Canvas-based treemap with cushion shading, selection borders, hover highlights
- **Views/TreeMap/TreeMapLayout.swift** - Squarified treemap algorithm with row-based layout, minProportion=0.4, maxDepth=30
- **Views/FileList/FileListView.swift** - Outline view with DisclosureGroups, auto-expands to selection
- **Views/Sidebar/SidebarView.swift** - File list + statistics summary

### Utilities

- **Utilities/FileSizeFormatter.swift** - Human-readable file size formatting (KB, MB, GB, TB)
- **Utilities/FileKindColorAssigner.swift** - Assigns colors to file kinds with normalization (R+G+B=1.8) to prevent dark boxes

## Key Patterns

- **SwiftUI + Observable** - `@Observable` macro for reactive state
- **Actor isolation** - FileScanner actor for thread-safe scanning
- **Structured concurrency** - Parallel directory scanning with task groups
- **Memory optimization** - ObjectIdentifier instead of UUID, UInt16 kindId instead of String
- **Color normalization** - BASE_BRIGHTNESS=1.8, redistributes overflow to prevent black boxes
- **Layout caching** - TreeMapView caches layout results to avoid recalculation on every draw

## Performance

- Scans 3.4M files in ~34 seconds
- Memory: ~260 bytes/node (down from ~500)
- Parallel scanning with batched concurrency (8 directories at a time)
- Statistics calculated in background after scan completes

## macOS Considerations

- Minimum deployment: macOS 14.0+
- Apple Silicon native (arm64)
- Requires Full Disk Access for scanning protected directories
- App Sandbox enabled with file read permissions

## Release Process

```bash
# Tag and push
git tag -a v1.0 -m "Version 1.0"
git push origin main --tags

# Create GitHub release
gh release create v1.0 "DiskInventoryY-1.0.dmg" \
  --title "Disk Inventory Y 1.0" \
  --notes "Release notes here"
```

## License

Copyright 2026 Mahmoud Lababidi. GPL v3

---
> Source: [diskinv/diy](https://github.com/diskinv/diy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
