## recipemd-swift

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Project: RecipeMD-swift

## Quick Reference
- **Platform**: iOS 17+ / macOS 14+
- **Language**: Swift 6.0
- **Architecture**: MVVM with @Observable
- **Minimum Deployment**: iOS 17.0
- **Package Manager**: Swift Package Manager

## XcodeBuildMCP Integration
**IMPORTANT**: This project uses XcodeBuildMCP for all Xcode operations.
- Build: `mcp__xcodebuildmcp__build_sim_name_proj`
- Test: `mcp__xcodebuildmcp__test_sim_name_proj`
- Clean: `mcp__xcodebuildmcp__clean`

## Project Structure
RecipeMD-swift
├── Library/ # Library entry point, App delegate
├── Core/ # Shared utilities
│ ├── Extensions/
│ ├── Services/
│ └── Networking/
├── Resources/ # Assets, Localizations

## Coding Standards

### Swift Style
- Use Swift 6 strict concurrency
- Prefer `@Observable` over `ObservableObject`
- Use `async/await` for all async operations
- Follow Apple's Swift API Design Guidelines
- Use `guard` for early exits
- Prefer value types (structs) over reference types (classes)

## Error Handling
```
// Always use typed errors
enum AppError: LocalizedError {
    case networkError(underlying: Error)
    case validationError(message: String)
    
    var errorDescription: String? {
        switch self {
        case .networkError(let error): return error.localizedDescription
        case .validationError(let msg): return msg
        }
    }
}
```

## Testing Requirements
- Use Swift Testing framework (@Test, #expect)
- Minimum 80% code coverage for business logic

## DO NOT
- Write UITests during scaffolding phase
- Use deprecated APIs (UIKit when SwiftUI suffices)
- Create massive monolithic views
- Use force unwrapping (!) without justification
- Ignore Swift 6 concurrency warnings

## Planning Workflow

When starting new features:
- Read the PRD from Docs/PRD.md
- Create feature spec in docs/specs/[feature-name].md
- Use ultrathink for architectural decisions
- Use Plan Mode (Shift+Tab) for implementation strategy
- Implement incrementally with tests

---
> Source: [davidbettis/recipemd-swift](https://github.com/davidbettis/recipemd-swift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
