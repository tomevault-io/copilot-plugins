## path-pal-mac-app

> Open-source macOS alternative to Default Folder X. Enhances Open/Save dialogs with quick folder navigation.

# PathPal

Open-source macOS alternative to Default Folder X. Enhances Open/Save dialogs with quick folder navigation.

## Architecture

- **Menu bar app** — NSStatusItem with cascading NSMenu, no dock icon (LSUIElement)
- **Non-sandboxed** — needs Accessibility + Automation permissions
- **SwiftUI + AppKit hybrid** — SwiftUI for views, NSPanel for overlay windows
- **Services pattern** — each feature is a standalone service initialized by AppDelegate

## Key Features

1. **Menu bar recent folders/files** — MenuBarService + RecentItemsService
2. **Open/Save dialog enhancement** — AccessibilityService detects dialogs → OverlayWindowService shows panel → DialogNavigationService navigates via Cmd+Shift+G keystroke simulation
3. **Path bar (Cmd+L)** — HotKeyService (Carbon) → PathBarPanel → FinderScriptingService navigates
4. **Finder window highlighting** — FinderWindowService (CGWindowList + AppleScript) → HighlightWindow overlays
5. **FinderSync extension** — Toolbar button in Finder, communicates via App Group UserDefaults
6. **Launch at login** — SMAppService (macOS 13+)

## Project Structure

```
PathPal/
├── PathPal.xcodeproj/
├── PathPal/              # Main app target
│   ├── PathPalApp.swift  # @main entry point
│   ├── AppDelegate.swift # Service initialization
│   ├── AppState.swift    # @Observable state
│   ├── Services/         # Feature services
│   ├── Models/           # Data models
│   ├── Views/            # SwiftUI views
│   ├── Windows/          # NSPanel subclasses
│   └── Resources/
├── PathPalTests/         # Unit tests
└── PathPalFinderExtension/  # FinderSync extension
```

## Build & Test

```bash
make install    # Build + install to /Applications + launch
make test       # Run unit tests
make clean      # Clean build artifacts
```

## Dialog Navigation Flow

1. AccessibilityService observes `kAXWindowCreatedNotification` on all apps
2. When a window appears, scan for "Open"/"Save"/"Upload" buttons → classify dialog type
3. OverlayWindowService creates an NSPanel with `worksWhenModal=true`, level `.modalPanel+1`
4. Panel shows: Finder windows, Quick Access (Desktop/Documents/Downloads), Recent folders
5. On folder selection: DialogNavigationService sends `Cmd+Shift+G` → types path → presses Return

## Permissions

- **Accessibility**: `AXIsProcessTrusted()` — needed for AXObserver and CGEvent keystroke posting
- **Automation**: AppleScript to Finder — needed for reading window paths and navigation

## Manual Testing Checklist

- [ ] Menu bar icon appears on launch
- [ ] Recent Folders submenu populates after visiting Finder windows
- [ ] Submenus show folder children on hover
- [ ] Settings window opens and toggles persist
- [ ] Launch at login toggle works
- [ ] Open TextEdit > File > Open → overlay appears
- [ ] Click Finder window entry → dialog navigates
- [ ] Click Desktop/Documents/Downloads → dialog navigates
- [ ] Close dialog → overlay dismisses
- [ ] Cmd+L in Finder → path bar appears
- [ ] Type path → autocomplete shows
- [ ] Enter → Finder navigates
- [ ] Escape → path bar dismisses

---
> Source: [NSEvent/path-pal-mac-app](https://github.com/NSEvent/path-pal-mac-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
