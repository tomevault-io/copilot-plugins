## task-management

> Auto-generated from all feature plans. Last updated: 2026-02-13

# task-management Development Guidelines

Auto-generated from all feature plans. Last updated: 2026-02-13

## Active Technologies
- Go 1.25.0 + Bubble Tea v1.3.10 (TUI), Bubbles v1.0.0 (components), Huh v0.8.0 (forms), Lip Gloss v1.1.0 (styling), sqlx v1.4.0 (database), modernc.org/sqlite v1.45.0 (002-todo-external-sources)
- SQLite (existing, extended with new tables for todos, projects, tags, checklist items, links) (002-todo-external-sources)
- Swift 6.x (latest stable) + SwiftUI, SwiftData, Foundation, Security (Keychain), AppKit (NSWorkspace), Accessibility APIs (004-macos-todo-app)
- SwiftData (macOS 14+) (004-macos-todo-app)
- Swift 6.0 (latest stable) + SwiftUI, SwiftData, ApplicationServices (AXUIElement), IOKit (idle detection), AppKit (NSWorkspace, NSAppleScript), Foundation, Security (Keychain) (005-app-time-tracking)
- SwiftData (SQLite-backed, macOS 14+) — extending existing 004 schema (005-app-time-tracking)
- Swift 6.0 (latest stable) + SwiftUI, SwiftData, ApplicationServices (AXUIElement), IOKit, AppKit (NSWorkspace, NSAppleScript), Foundation, Security (Keychain) (006-time-tracking-plugins)
- SwiftData (SQLite-backed, macOS 14+) — extending existing schema (006-time-tracking-plugins)

- Go 1.22+ + Bubble Tea v1.x (TUI), Lip Gloss v1.x (styling), Bubbles v0.20+ (components), Huh v0.6+ (forms), Glamour (markdown rendering) (001-terminal-task-manager)

## Project Structure

```text
src/
tests/
```

## Commands

# Add commands for Go 1.22+

## Code Style

Go 1.22+: Follow standard conventions

## Recent Changes
- 006-time-tracking-plugins: Added Swift 6.0 (latest stable) + SwiftUI, SwiftData, ApplicationServices (AXUIElement), IOKit, AppKit (NSWorkspace, NSAppleScript), Foundation, Security (Keychain)
- 005-app-time-tracking: Added Swift 6.0 (latest stable) + SwiftUI, SwiftData, ApplicationServices (AXUIElement), IOKit (idle detection), AppKit (NSWorkspace, NSAppleScript), Foundation, Security (Keychain)
- 004-macos-todo-app: Added Swift 6.x (latest stable) + SwiftUI, SwiftData, Foundation, Security (Keychain), AppKit (NSWorkspace), Accessibility APIs


<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nam-hle)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nam-hle)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
