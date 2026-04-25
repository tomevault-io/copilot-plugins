## spotify-story-teller

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Test Commands
- Build: Open in Xcode and use Product > Build (⌘B)
- Run: Use Product > Run (⌘R)
- Test: Use Product > Test (⌘U)
- Run single test: Use the test diamond icon in Xcode test file gutter

## Code Style Guidelines
- **Imports**: Group imports with Foundation/SwiftUI first, then alphabetically
- **Formatting**: 4-space indentation, trailing commas in multi-line collections
- **Types**: Use explicit type annotations for public APIs, infer when obvious
- **Naming**: Use camelCase for variables/functions, PascalCase for types
- **Error Handling**: Use `do/catch` blocks with descriptive error messages
- **SwiftUI**: Use property wrappers (@State, @EnvironmentObject) appropriately
- **Async Code**: Use Combine for reactive data flows, Task for async/await
- **Comments**: Use // for single line comments, /// for documentation comments
- **Environment Variables**: Access via Environment.shared
- **Resources**: Store API keys in .env file (not in git)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LoopKarma) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
