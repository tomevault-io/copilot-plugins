## cctray

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Development Commands

**Building the app:**
```bash
# Open in Xcode and build with Cmd+B, or use command line:
xcodebuild -project CCTray.xcodeproj -scheme CCTray -configuration Debug
```

**Running tests:**
```bash
# Unit tests
xcodebuild test -project CCTray.xcodeproj -scheme CCTray -destination 'platform=macOS'

# In Xcode: Cmd+U for all tests
```

**Running the app:**
- Open `CCTray.xcodeproj` in Xcode and press Cmd+R
- App appears as orange "C" icon in macOS menu bar (no dock icon)

## Architecture Overview

CCTray is a **macOS menu bar application** that monitors Claude Code usage through the `ccusage` CLI command. Built with SwiftUI using modern menu bar patterns.

**Key Architecture Patterns:**
- **MVVM**: Models in `Models/`, Views in `Views/`, ViewModels in `ViewModels/`
- **State Management**: `@StateObject`, `@EnvironmentObject`, and `@AppStorage` for persistence
- **Dependency Injection**: Environment objects passed through SwiftUI hierarchy
- **Actor-based Services**: `CommandExecutor` uses Swift actors for safe async operations

**Core Components:**
- `CCTrayApp.swift`: Entry point using `MenuBarExtra` and `Settings` scene
- `UsageMonitor`: Main business logic, fetches data every 5 seconds via `ccusage` command
- `AppPreferences`: User settings with `@AppStorage` persistence
- `CommandExecutor`: Executes shell commands with proper PATH resolution for Node.js

**External Dependencies:**
- Requires `ccusage` CLI tool (Node.js based)
- Default path: `~/.nvm/versions/node/v20.11.0/lib/node_modules/ccusage/dist/index.js`
- **Node.js Detection**: Automatically searches common installation paths:
  - Homebrew: `/opt/homebrew/bin/node`
  - System: `/usr/local/bin/node`, `/usr/bin/node`
  - nvm: `~/.nvm/versions/node/*/bin/node`
  - Also checks current PATH environment variable

## Key Technical Details

**Menu Bar Integration:**
- Uses modern `MenuBarExtra` (requires macOS 13.0+)
- `LSUIElement = YES` in Info.plist (menu bar only, no dock icon)
- Custom orange "C" icon generated programmatically in `IconCreator.swift`

**Data Flow:**
- `UsageMonitor` executes `node ccusage` command via `CommandExecutor`
- JSON response parsed into `CCUsageResponse` model
- UI updates automatically via `@Published` properties
- Rotating display: Cost → Burn Rate → Remaining Time

**State Persistence:**
- User preferences stored via `@AppStorage` in `AppPreferences.swift`
- Settings include update intervals, display toggles, custom command paths
- Three-tab preferences window: General/Display/Advanced

**Error Handling:**
- Command execution timeouts (30 seconds)
- Specific error messages for permission denied (exit code 126) and command not found (127)
- Direct executable path resolution (bypasses `/usr/bin/env` for security)
- Graceful fallbacks for missing `ccusage` command
- Manual PATH resolution for different Node.js installations

## Important Configuration

**Deployment Target:** macOS 15.5 (current), minimum macOS 13.0 for MenuBarExtra support

**Info.plist Settings:**
- `LSUIElement = YES` (required for menu bar only behavior)
- `NSApplicationShouldTerminateAfterLastWindowClosed = NO`

**Entitlements (CCTray.entitlements):**
- `com.apple.security.app-sandbox = false` (disabled to allow command execution)
- `com.apple.security.files.user-selected.read-only = true` (file access)

**File Organization:**
- Main app source in `CCTray/CCTray/`
- Organized by purpose: Models, Views, ViewModels, Services, Utilities
- Tests in `CCTrayTests/` and `CCTrayUITests/`

When modifying this app, maintain the existing MVVM architecture and use SwiftUI's reactive patterns. All external command execution should go through `CommandExecutor` for proper async handling.

---
> Source: [goniszewski/cctray](https://github.com/goniszewski/cctray) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
