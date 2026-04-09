## catpaws

> A macOS menu bar app that detects when cats walk on your keyboard and temporarily locks input.

# CatPaws Development Guidelines

A macOS menu bar app that detects when cats walk on your keyboard and temporarily locks input.

## Technologies

- **Swift 5.9+** with **Xcode 15+**
- **SwiftUI** for UI views
- **AppKit**: NSEvent (global keyboard monitoring), NSStatusItem (menu bar), NSPanel (floating notifications), NSWindow
- **Accessibility APIs (CGEvent)** for input blocking
- **ServiceManagement (SMAppService)** for launch at login
- **Carbon (TISGetInputSourceProperty)** for keyboard layout detection
- **UserDefaults** for configuration and statistics persistence
- **XCTest** for unit, integration, and UI tests

## Project Structure

```text
CatPaws/
├── CatPaws/                    # Xcode project
│   ├── CatPaws/                # Source code
│   │   ├── App/                # App entry point, AppDelegate
│   │   ├── Models/             # Data models (AppState, Configuration, etc.)
│   │   ├── Services/           # Core services (detection, locking, monitoring)
│   │   ├── Views/              # SwiftUI views
│   │   ├── ViewModels/         # MVVM view models
│   │   ├── MenuBar/            # Menu bar UI
│   │   └── Configuration/      # Info.plist, entitlements
│   ├── CatPawsTests/           # Unit & integration tests
│   └── CatPawsUITests/         # UI tests
├── CatPaws_Website/            # Landing page website
├── scripts/                    # Build scripts
└── specs/                      # Feature specifications
```

## Commands

```bash
# Build (from CatPaws/ directory)
xcodebuild -project CatPaws.xcodeproj -scheme CatPaws -configuration Debug build

# Run tests
xcodebuild -project CatPaws.xcodeproj -scheme CatPaws test

# Build release DMG (signed & notarized)
./scripts/build-release.sh

# Lint
swiftlint --config CatPaws/.swiftlint.yml
```

## Architecture

The app uses **MVVM** architecture:
- **Models**: Pure data types (`AppState`, `Configuration`, `KeyboardState`, `LockState`)
- **Services**: Business logic with protocols for testability (`CatDetectionService`, `KeyboardMonitor`, `KeyboardLockService`, `LockStateManager`)
- **ViewModels**: `AppViewModel`, `OnboardingViewModel` bridge services to views
- **Views**: SwiftUI views (`SettingsView`, `OnboardingView`, `CatLockPopupView`, `StatisticsView`)

## Code Style

- SwiftLint configured in `CatPaws/.swiftlint.yml`
- Protocol-based design for testability (services implement protocols like `CatDetecting`, `KeyboardLocking`, `KeyboardMonitoring`)
- Test mocks in `CatPawsTests/Mocks/`

## Key Files

- `CatPaws/CatPaws/Services/CatDetectionService.swift` - Cat detection algorithm
- `CatPaws/CatPaws/Services/KeyboardMonitor.swift` - Global keyboard event capture
- `CatPaws/CatPaws/Services/KeyboardLockService.swift` - Input blocking via CGEvent
- `CatPaws/CatPaws/Services/LockStateManager.swift` - Lock state coordination
- `CatPaws/CatPaws/ViewModels/AppViewModel.swift` - Main app state management


<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TechPreacher)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/TechPreacher)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
