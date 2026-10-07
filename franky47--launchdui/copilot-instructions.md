## launchdui

> - After implementing any task, update `WIP.md` to mark it complete (`[x]`) and update the "Current Focus" section.

# LaunchdUI Project Instructions

## WIP Tracking
- After implementing any task, update `WIP.md` to mark it complete (`[x]`) and update the "Current Focus" section.
- Keep WIP.md in sync with actual project state at all times.

## Architecture
- Swift 6 strict concurrency. macOS 15+. SwiftUI.
- Built as a Swift Package (no Xcode project).
- **Strictly read-only** — never execute mutating `launchctl` commands.
- `AppState` is `@MainActor @Observable`. `ServiceRepository` is an `actor`.
- Parsers are pure static functions.
- Use `PlistValue` (recursive Sendable enum) to cross isolation boundaries.

## Code Style
- No Xcode project files — use `swift build` / `swift test` / `swift run`.
- Prefer explicit types at API boundaries.
- Keep files focused — one primary type per file.

## Testing
- Parser tests use fixture strings (captured real output).
- Tests live in `Tests/LaunchdUITests/`.

---
> Source: [franky47/launchdui](https://github.com/franky47/launchdui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
