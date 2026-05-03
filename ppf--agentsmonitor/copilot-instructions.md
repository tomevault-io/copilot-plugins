## agentsmonitor

> AgentsMonitor is a native macOS SwiftUI application for monitoring AI coding agent sessions (Claude Code, Codex). It provides an embedded terminal, tool call timeline, token metrics, and session management.

# Repository Guidelines

## Project Overview

AgentsMonitor is a native macOS SwiftUI application for monitoring AI coding agent sessions (Claude Code, Codex). It provides an embedded terminal, tool call timeline, token metrics, and session management.

**Platform:** macOS 14.0+ (Sonoma)
**Language:** Swift 5.9+
**UI Framework:** SwiftUI with AppKit bridges
**Architecture:** MVVM with `@Observable`

## Project Structure & Module Organization

- `AgentsMonitor/` contains the macOS SwiftUI app source.
- `AgentsMonitor/App/` holds the app entry point (`AgentsMonitorApp.swift`).
- `AgentsMonitor/Models/` holds data types: `Session`, `Message`, `ToolCall`, `AppState`, `AppEnvironment`.
- `AgentsMonitor/ViewModels/SessionStore.swift` is the single source of truth for all session state.
- `AgentsMonitor/Services/` holds actor-based services: `AgentService`, `AgentProcessManager`, `SessionPersistence`, `TerminalBridge`, `Logger`.
- `AgentsMonitor/Views/` holds all SwiftUI views organized by feature area.
- `AgentsMonitor/Components/` holds reusable UI components like `StatusBadge`.
- `AgentsMonitor/Theme/` holds `AppTheme.swift` (colors, fonts, spacing) and `TerminalThemes.swift`.
- `AgentsMonitor/Resources/` holds `Assets.xcassets`, fonts, and `Info.plist`.
- `AgentsMonitorTests/` contains XCTest unit tests (80+ tests).
- `AgentsMonitorUITests/` contains UI integration tests.

## Build, Test, and Development Commands

```bash
# Open in Xcode
open AgentsMonitor/AgentsMonitor.xcodeproj

# Build from CLI
xcodebuild build -project AgentsMonitor/AgentsMonitor.xcodeproj -scheme AgentsMonitor -destination "platform=macOS"

# Run all tests from CLI (requires full Xcode install)
xcodebuild test -project AgentsMonitor/AgentsMonitor.xcodeproj -scheme AgentsMonitor -destination "platform=macOS"

# Run a specific test class
xcodebuild test -project AgentsMonitor/AgentsMonitor.xcodeproj -scheme AgentsMonitor -destination "platform=macOS" -only-testing:AgentsMonitorTests/SessionStoreTests

# Run a single test method
xcodebuild test -project AgentsMonitor/AgentsMonitor.xcodeproj -scheme AgentsMonitor -destination "platform=macOS" -only-testing:AgentsMonitorTests/SessionStoreTests/testCreateNewSession
```

In Xcode: Build (Cmd+B), Run (Cmd+R), Test (Cmd+U).

## Key Architecture Patterns

### State Management
- `SessionStore` is an `@Observable` class -- the single source of truth.
- Views inject it via `@Environment(SessionStore.self)`.
- Use `@Bindable var store = sessionStore` for two-way binding.
- **Do NOT** use `@StateObject` or `ObservableObject` -- this project uses Swift 5.9+ `@Observable`.

### Actor-Based Concurrency
- `AgentService`, `SessionPersistence`, `AgentProcessManager` are Swift actors.
- No manual locking -- actor isolation provides thread safety.

### Dependency Injection
```swift
// Production
SessionStore(agentService: AgentService(), persistence: SessionPersistence.shared)

// Tests -- nil persistence loads deterministic mock data
SessionStore(agentService: AgentService(), persistence: nil)
```

### Data Flow
```
AgentProcessManager (spawn/signal) -> TerminalBridge (I/O)
    |
SessionStore (@Observable) -> State mutations + cache invalidation
    |
Views (@Environment) -> Reactive UI updates
    |
SessionPersistence (actor) -> JSON files on disk
```

## Coding Style & Naming Conventions

- Swift 4-space indentation; follow Swift API Design Guidelines.
- Types use `PascalCase`, variables/functions use `camelCase`.
- File names match primary types (e.g., `SessionStore.swift`).
- Keep SwiftUI views small; compose via subviews.
- Use `AppTheme` for all colors, fonts, spacing -- never hardcode colors in views.
- Use `AppLogger` for logging -- never use `print()`.
- All icon buttons must have `.accessibilityLabel()` and `.accessibilityHint()`.

## Testing Guidelines

- XCTest is used (`import XCTest`, `@testable import AgentsMonitor`).
- Test files follow `*Tests.swift`, classes end with `Tests`.
- Test methods use `test...` naming.
- `SessionStore` tests must use `@MainActor` since the store performs UI-bound mutations.
- Use `persistence: nil` in test setup to avoid disk I/O and load deterministic mock data.
- Wait for mock data loading: `try await Task.sleep(nanoseconds: 200_000_000)` after creating the store.
- Keep tests deterministic; prefer dependency injection.

## Important Conventions When Modifying Code

- Call `invalidateCache()` after any mutation to the `sessions` array in `SessionStore`.
- Call `persistSession()` after state changes to save to disk asynchronously.
- Terminal output is capped at 10 MB per session; persistence writes are throttled to 500ms intervals.
- The `exitCode` field on `Session` stores the process exit code for post-mortem analysis.
- When adding a new `SessionStatus`, update: the enum, `AppTheme.statusColors`, `AppState.SessionFilter`, `SessionStore.filteredSessions()`, and `SessionListView` FilterMenu.

## Commit & Pull Request Guidelines

- Use short, imperative, capitalized commit subjects.
  - Example: `Fix terminal output memory leak and add persistence throttling`
- PRs should include:
  - A clear summary of changes.
  - Testing notes (commands run or "not run" with reason).
  - Screenshots for UI changes.

## Security & Configuration

- Do not hardcode secrets; use environment variables or local config.
- Avoid logging sensitive data (tokens, credentials, PII).
- App entitlements live in `AgentsMonitor/AgentsMonitor.entitlements`.
- Override the sessions directory with env var: `AGENTS_MONITOR_SESSIONS_DIR=/path/to/dir`

---
> Source: [ppf/agentsmonitor](https://github.com/ppf/agentsmonitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
