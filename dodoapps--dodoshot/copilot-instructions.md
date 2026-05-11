## dodoshot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

Open in Xcode and build with ⌘R:
```bash
open DodoShot.xcodeproj
```

Command-line build:
```bash
xcodebuild -project DodoShot.xcodeproj -scheme DodoShot -configuration Debug build
```

Archive for release:
```bash
xcodebuild -project DodoShot.xcodeproj -scheme DodoShot -configuration Release archive -archivePath build/DodoShot.xcarchive
xcodebuild -exportArchive -archivePath build/DodoShot.xcarchive -exportPath release/ -exportOptionsPlist exportOptions.plist
```

**IMPORTANT:** Always reset permissions when deploying a new build to test the onboarding flow:
```bash
tccutil reset ScreenCapture com.dodoshot.app
tccutil reset Accessibility com.dodoshot.app
```

There are no unit tests in this project. Requires Xcode 15+, macOS 14.0+ (Sonoma), Swift 5.9.

## Architecture

Native macOS menu bar app built with SwiftUI + AppKit. Runs as a menu bar item (NSStatusItem) with left-click popover and right-click context menu. Uses `NSApplicationDelegateAdaptor` in `DodoShotApp.swift` to bridge SwiftUI app lifecycle with AppKit window management.

### Singleton services (all use `static let shared`)

- **ScreenCaptureService** — Central capture orchestrator. Manages area/window/fullscreen/scrolling/OCR/timed captures. Creates overlay windows per-screen, captures via `CGWindowListCreateImage`, and routes completed captures to the annotation editor.
- **SettingsManager** — Persists `AppSettings` to UserDefaults as JSON. Settings auto-save on `didSet` and auto-apply appearance mode.
- **HotkeyManager** — Global keyboard shortcuts via `CGEvent` tap (requires Accessibility permission).
- **FloatingWindowService** — Manages pinned always-on-top screenshot windows with opacity/click-through controls.
- **PermissionManager** — Checks Screen Recording and Accessibility permissions.
- **ScrollingCaptureService** — Auto-scrolls a window and stitches captures.
- **OCRService** — Text extraction using Apple Vision framework (on-device, no API key).
- **LLMService** — Optional AI descriptions via Anthropic or OpenAI API.

### Window controllers (singleton pattern)

- **AnnotationEditorWindowController** — Opens annotation editor for a screenshot. Entry point after every capture completes. Defined in `AnnotationEditorView.swift`.
- **QuickOverlayManager** — Stacking corner overlays (CleanShot X style) with auto-dismiss. Defined in `QuickOverlayView.swift`.
- **PermissionOnboardingWindowController** — First-launch permission request flow. Defined in `PermissionView.swift`.

### Key design decisions

- **Screenshot stores image as `Data` (PNG bytes), not `NSImage`** — This is a struct (value type) that avoids use-after-free crashes. Each access to `Screenshot.image` returns a fresh `NSImage`. No `deepCopy()` needed.
- **Capture flow:** `ScreenCaptureService.completeCapture()` converts to PNG data once, then passes raw data (not NSImage references) to the editor via `openEditorDirectly()`.
- **Coordinate systems:** Area capture uses top-left origin matching `CGWindowListCreateImage`. Screen offset is added for multi-monitor. Retina scaling: always use point sizes (not pixel sizes) when creating `NSImage` from `CGImage`.
- **CaptureWindow** (subclass of NSWindow) — Intercepts ESC key at three levels (`keyDown`, `cancelOperation`, `performKeyEquivalent`) to prevent app termination during capture.

### File format

`.dodo` project files are JSON-encoded `DodoShotProject` structs containing PNG image data + annotations. Backward-compatible decoding uses `decodeIfPresent` with defaults for newer fields.

## Required macOS permissions

- **Screen Recording** (`ScreenCapture`) — For `CGWindowListCreateImage`
- **Accessibility** — For global hotkey event tap via `AXIsProcessTrusted()`

The app polls for permission changes every 2 seconds until both are granted.

## Code conventions

- Swift API Design Guidelines, SwiftUI for all views
- `@MainActor` on observable service classes
- MARK comments for section organization
- Bundle ID: `com.dodoshot.app`

---
> Source: [DodoApps/dodoshot](https://github.com/DodoApps/dodoshot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
