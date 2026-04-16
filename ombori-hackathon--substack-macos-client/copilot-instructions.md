## substack-macos-client

> macOS desktop app that communicates with the FastAPI backend.

# SubStackClient - SwiftUI App

macOS desktop app that communicates with the FastAPI backend.

## Commands
- Build: `swift build`
- Run: `swift run SubStackClient` (opens GUI window)
- Test: `swift test`

## Skills (Commands)
Available in `.claude/skills/`:
- `/feature` - Build new SwiftUI features with TDD
- `/test` - Run Swift tests

## Agents
Available in `.claude/agents/`:
- `swift-coder` - SwiftUI development
- `tester` - Write and run XCTest tests
- `reviewer` - Code review before commits
- `debugger` - Investigate Swift/SwiftUI issues

## Architecture
- SwiftUI app with native macOS window
- Entry point: Sources/SubStackApp.swift
- Main view: Sources/ContentView.swift
- Data models: Sources/Models.swift
- Uses async/await with URLSession
- Targets macOS 14+

## API Integration
- Backend runs at http://localhost:8000
- Health check: GET /health
- Sample data: GET /items (returns list of items)

## Adding Features
1. Create new SwiftUI views in Sources/
2. Add new async functions for API calls in views or a dedicated APIClient
2. Use `URLSession.shared.data(from:)` for GET requests
3. Use `URLSession.shared.data(for:)` for POST/PUT with URLRequest

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ombori-hackathon) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
