## screencapture

> ScreenCapture is a fast, lightweight macOS menu bar application for capturing and annotating screenshots. It uses Apple's ScreenCaptureKit API and provides annotation tools similar to professional screenshot utilities.

# CLAUDE.md - ScreenCapture

## Project Overview

ScreenCapture is a fast, lightweight macOS menu bar application for capturing and annotating screenshots. It uses Apple's ScreenCaptureKit API and provides annotation tools similar to professional screenshot utilities.

**Tech Stack:** Swift 6.2, SwiftUI + AppKit, ScreenCaptureKit, CoreGraphics
**Minimum macOS:** 13.0 (Ventura)
**Build System:** Xcode 15.0+

## Build & Run

```bash
# Open in Xcode and run (Cmd+R)
open ScreenCapture.xcodeproj

# Command line build
xcodebuild -project ScreenCapture.xcodeproj -scheme ScreenCapture

# Archive for distribution
xcodebuild archive -project ScreenCapture.xcodeproj -scheme ScreenCapture
```

**Required Permission:** Screen Recording (System Settings → Privacy & Security → Screen Recording)

## Project Structure

```
ScreenCapture/
├── App/                                    # Application entry point
│   ├── ScreenCaptureApp.swift             # @main SwiftUI app with AppDelegate adaptor
│   └── AppDelegate.swift                  # Central coordinator: lifecycle, hotkeys, capture flow
│
├── Features/
│   ├── Capture/                           # Screenshot capture system
│   │   ├── CaptureManager.swift           # Actor: thread-safe ScreenCaptureKit operations
│   │   ├── ScreenDetector.swift           # Actor: display enumeration with 5s caching
│   │   ├── SelectionOverlayWindow.swift   # Region selection UI overlay on all displays
│   │   └── DisplaySelector.swift          # Multi-monitor display selection
│   │
│   ├── Preview/                           # Screenshot editing/annotation
│   │   ├── PreviewWindow.swift            # NSPanel window for screenshot editing
│   │   ├── PreviewViewModel.swift         # @Observable state: tools, annotations, undo
│   │   ├── PreviewContentView.swift       # SwiftUI content view with toolbar
│   │   └── AnnotationCanvas.swift         # Drawing surface for annotation rendering
│   │
│   ├── Annotations/                       # Drawing tools system
│   │   ├── AnnotationTool.swift           # Protocol: beginDrawing, continueDrawing, endDrawing
│   │   ├── RectangleTool.swift            # Rectangle outlines with configurable stroke
│   │   ├── FreehandTool.swift             # Freehand drawing paths
│   │   ├── ArrowTool.swift                # Directional arrows
│   │   └── TextTool.swift                 # Text annotations with font/color options
│   │
│   ├── MenuBar/                           # Status bar integration
│   │   └── MenuBarController.swift        # @MainActor: status item + dropdown menu
│   │
│   └── Settings/                          # Preferences UI
│       ├── SettingsView.swift             # SwiftUI settings form
│       ├── SettingsViewModel.swift        # @Observable settings state
│       └── SettingsWindowController.swift # NSWindow controller for preferences
│
├── Services/                              # Business logic layer
│   ├── ImageExporter.swift                # PNG/JPEG encoding, save to disk/clipboard
│   ├── HotkeyManager.swift                # Actor: global hotkey registration (Carbon APIs)
│   ├── ClipboardService.swift             # NSPasteboard operations
│   └── RecentCapturesStore.swift          # Recent captures history with thumbnails
│
├── Models/                                # Data types
│   ├── Screenshot.swift                   # Immutable: CGImage, metadata, annotations
│   ├── Annotation.swift                   # Enum: Rectangle, Freehand, Arrow, Text subtypes
│   ├── AppSettings.swift                  # @Observable singleton: UserDefaults preferences
│   ├── DisplayInfo.swift                  # Display metadata (resolution, scale, position)
│   ├── ExportFormat.swift                 # PNG/JPEG format enum with quality settings
│   ├── KeyboardShortcut.swift             # Hotkey representation (key + modifiers)
│   └── Styles.swift                       # StrokeStyle, TextStyle value types
│
├── Extensions/                            # Swift extensions
│   ├── CGImage+Extensions.swift           # CGImage helpers (cropping, scaling)
│   ├── NSImage+Extensions.swift           # NSImage ↔ CGImage conversions
│   └── View+Cursor.swift                  # SwiftUI cursor modifier
│
├── Errors/                                # Custom error types
│   └── ScreenCaptureError.swift           # Capture, permission, export errors
│
└── Resources/
    └── Assets.xcassets                    # App icon, colors, images
```

**File Count:** 34 Swift files across 11 directories

## Key Files

- **`App/AppDelegate.swift`** - Central coordinator for lifecycle, hotkeys, and capture flow
- **`Features/Capture/CaptureManager.swift`** - Actor for thread-safe ScreenCaptureKit operations
- **`Features/Capture/SelectionOverlayWindow.swift`** - Region selection UI
- **`Features/Preview/PreviewViewModel.swift`** - @Observable state management for annotations
- **`Features/Annotations/AnnotationTool.swift`** - Protocol for annotation tools
- **`Services/ImageExporter.swift`** - PNG/JPEG encoding and save operations
- **`Services/HotkeyManager.swift`** - Global hotkey registration (Carbon APIs)
- **`Models/Screenshot.swift`** - Immutable screenshot model with annotations
- **`Models/AppSettings.swift`** - Observable singleton for UserDefaults preferences

## Architecture

**Threading Model:**
- `@MainActor` for all UI components (AppDelegate, MenuBarController, windows)
- `actor` for thread-safe operations (CaptureManager, ScreenDetector, HotkeyManager)
- Background tasks for image encoding and file I/O

**Patterns Used:**
- Actor-based concurrency (Swift 6 strict concurrency)
- MVVM with @Observable macro
- Protocol-based design for annotation tools
- Feature-based module organization

**Application Flow:**
```
Hotkey pressed → AppDelegate → CaptureManager/SelectionOverlay
    → Screenshot created → PreviewWindow with PreviewViewModel
    → User annotates → ImageExporter → Save/Clipboard
```

## Annotation Tools

Four tools implementing `AnnotationTool` protocol:
- **RectangleTool** - Rectangle outlines with configurable stroke
- **FreehandTool** - Freehand drawing paths
- **ArrowTool** - Directional arrows
- **TextTool** - Text annotations with font/color options

## Code Style

- Swift 6 with strict concurrency checking enabled
- Use `actor` for thread-safe shared state
- Use `@Observable` macro for SwiftUI state management
- Prefer `struct` and `final class`
- DocC-style comments for public APIs
- Conventional commits (feat:, fix:, docs:, refactor:, test:)

## Testing

No automated tests currently. Manual testing checklist:
- Full screen capture (primary and secondary displays)
- Selection region capture
- All 4 annotation tools
- Undo/redo functionality
- Crop mode
- Save to disk and clipboard
- Settings persistence
- Hotkey registration
- Menu bar functionality

## Dependencies

No external dependencies. Uses only Apple frameworks:
- ScreenCaptureKit, AppKit, SwiftUI, CoreGraphics, Carbon (for hotkeys)

## Notes

- App runs as menu bar only (`.accessory` activation policy)
- Sandboxing disabled due to Carbon hotkey APIs
- Uses security-scoped bookmarks for file access
- Display cache validity: 5 seconds
- Undo stack limited to 50 states
- Recent captures limited to 5 entries

---
> Source: [sadopc/ScreenCapture](https://github.com/sadopc/ScreenCapture) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
