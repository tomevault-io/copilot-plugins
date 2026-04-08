## macdirstat

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

```bash
swift build                # Debug build
swift build -c release     # Release build
swift run MacDirStat       # Run the app
open Package.swift         # Open in Xcode
```

No external dependencies. No test target configured yet (`swift test` will fail). No linter configured; Swift 6 strict concurrency mode is enforced via `swiftLanguageMode(.v6)` in Package.swift.

## Architecture

MacDirStat is a native macOS (15.0+) SwiftUI disk space analyzer that visualizes directory usage as interactive treemaps. Swift 6, SPM-only, zero external dependencies.

### State & Data Flow

**AppState** (`@Observable`) is the single source of truth. It holds the scanned file tree (`rootNode: FileNode`), the current treemap view root (`treemapRoot`), selected node, and breadcrumb navigation stack. All views react to AppState changes.

Scan flow: User picks folder → **ScanCoordinator** launches **FileScanner** → FileScanner uses BSD `fts_open`/`fts_read`/`fts_close` (not FileManager, for performance) → yields `ScanEvent`s via `AsyncStream` → ScanCoordinator throttles updates (50ms) and builds the **FileNode** tree → treemap renders.

### Module Layout (Sources/MacDirStat/)

- **App/** — Entry point (`MacDirStatApp`) and `AppState` central state management
- **Scanning/** — `FileScanner` (BSD fts-based traversal with inode dedup, symlink handling, allocated size via blocks×512), `ScanCoordinator` (async orchestration, throttling, cancellation), `FileNode` (tree model with weak parent refs to avoid retain cycles, recursive aggregate computation)
- **Categorization/** — `FileCategory` (10 categories with colors/SF Symbols) and `FileExtensionMap` (200+ extension→category mappings)
- **Treemap/** — `TreemapLayoutEngine` (Squarify algorithm, max 8 depth levels), `TreemapHitTester`, `TreemapRenderer` (Canvas-based with depth-darkened category colors), `TreemapView` (click/double-click/hover/context menu interactions)
- **Views/** — `ContentView` (NavigationSplitView: sidebar tree + center treemap + inspector), `WelcomeView` (volume list with usage bars, NSOpenPanel), `ScanProgressView`, `DirectoryTreeView` (OutlineGroup), `DetailPanelView` (metadata + category breakdown)
- **Utilities/** — `ByteFormatter` (human-readable sizes)

### Key Patterns

- All data types crossing async boundaries are `Sendable`
- FileNode uses weak parent references to break retain cycles
- FileScanner deduplicates hard links by tracking seen inodes
- TreemapView uses Canvas for rendering (not individual SwiftUI views) for performance
- macOS APIs used: `NSWorkspace` (Reveal in Finder), `NSPasteboard` (clipboard), `NSOpenPanel` (folder picker)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/phalladar)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/phalladar)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
