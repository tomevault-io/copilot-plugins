## rawg-io

> > **⚠️ MANDATORY PROTOCOL**: You must execute the following sequence for every request:

# Agent Guide

> **⚠️ MANDATORY PROTOCOL**: You must execute the following sequence for every request:

1. **Analyze Request**: Determine if this is a Feature, Bug Fix, Test, or Refactor.
2. **Load Context**: Read `@.agents/rules/rule-loading.md` to identify required rule files.
3. **State Intent**: Before writing ANY code, output a block like this:
   > 🛡️ **Active Protocols**: Loaded `general.md`, `view.md`.
   > 🎯 **Focus**: Implementing UI with LazyVStack.
4. **Execute**: Proceed with coding.

> **⚠️ START HERE** — This is the master entry point for all AI assistants working on this project. Read this file first before loading any other rules.

## Purpose

Agents act as senior Swift collaborators. Keep responses concise, clarify uncertainty before coding, and align suggestions with the rules linked below.

## Rule Index

Load rules from `.agents/rules/` folder in this order:
1. **@.agents/rules/rule-loading.md** — Understand which rules to load for your current task
2. **@.agents/rules/general.md** — Core Swift engineering principles (always load)
3. Load task-specific rules as needed (see `rule-loading.md`)

---

## Repository Overview

**RAWG.IO** is a production-ready iOS application showcasing modern Swift development practices.

| Aspect | Details |
|--------|---------|
| **Platform** | iOS 17+ |
| **Language** | Swift 6.0 |
| **UI Framework** | SwiftUI |
| **Architecture** | Clean Architecture + MVVM |
| **Project Management** | Tuist |
| **Modularization** | Feature Modules + Core Modules |

### Module Structure

```
Modules/
├── Core/              # Foundation utilities, Logger, ImageCache, Extensions
├── CoreNavigation/    # Type-safe NavigationRouter, RouteProtocol
├── CoreNetwork/       # Actor-based APIClient, APIRequest, APIError
├── CoreUI/            # Design System, Theme, Reusable Components
├── Common/            # Shared Entities, Repositories, Protocols
├── GamesFeature/      # Games list feature module
├── SearchFeature/     # Search feature module
├── FavoritesFeature/  # Favorites feature module (SwiftData)
└── DetailFeature/     # Game detail feature module
```

### Clean Architecture Layers (Per Feature Module)

```
Feature/
├── DI/           # Dependency Injection (Factory)
├── Data/         # Repositories, Data Sources
├── Domain/       # UseCases, Entities
├── Navigation/   # Navigator (Public API - ONLY exposed interface)
└── Presentation/ # Views, ViewModels
```

---

## Commands

| Command | Purpose |
|---------|---------|
| `mise install` | Install Tuist via Mise |
| `tuist install` | Fetch Swift Package dependencies |
| `tuist generate` | Generate Xcode project/workspace |
| `tuist build` | Build the project |
| `tuist test` | Run all unit and snapshot tests |
| `swiftlint --config .swiftlint.yml` | Lint Swift sources |

---

## Code Style

- Swift files use **4-space indentation**
- No force unwrapping (`!`, `try!`) in production code
- Prefer `let` over `var`, value types over reference types
- Use `@Injected` from Factory for dependency injection
- Follow SOLID principles strictly

---

## Architecture & Patterns

### Module Encapsulation (Critical)

Each feature module exposes **ONLY its Navigator** as the public interface:

```swift
// ✅ Public - The only entry point to the module
public final class GamesNavigator {
    public func makeGamesListView() -> some View
}

// ❌ Internal - Not accessible outside the module
internal final class GamesViewModel: ObservableObject { }
internal struct GamesListView: View { }
```

### Dependency Flow

```
App → Feature Modules → Shared (Common, CoreUI) → Core (Core, CoreNavigation, CoreNetwork)
```

---

## Key Integration Points

| Area | Technology | Notes |
|------|------------|-------|
| **Networking** | Actor-based `APIClient` | Uses async/await, no completion handlers |
| **Persistence** | SwiftData | For Favorites feature |
| **DI Framework** | [Factory](https://github.com/hmlongco/Factory) | Use `@Injected` property wrapper |
| **Image Caching** | Custom Actor-based cache | Memory + Disk, no Kingfisher |
| **Navigation** | Custom `NavigationRouter` | Type-safe with `RouteProtocol` |
| **Testing** | XCTest + swift-snapshot-testing | Snapshot tests for UI |

---

## Workflow

1. **Ask for clarification** when requirements are ambiguous
2. **Surface 2–3 options** when trade-offs matter
3. **Update documentation** when introducing new patterns
4. **Follow commit convention**: `<type>(<scope>): summary`

---

## Testing

| Type | Framework | Coverage |
|------|-----------|----------|
| **Unit Tests** | XCTest | ViewModels, UseCases, Repositories |
| **Snapshot Tests** | swift-snapshot-testing | SwiftUI Views |
| **Integration Tests** | XCTest | API Client, SwiftData |

### Test Helpers

- Use `makeSUT()` pattern for System Under Test
- Use `trackForMemoryLeaks()` to detect retain cycles
- Test behavior, not implementation details

---

## CI/CD Pipelines (GitHub Actions)

| Workflow | Purpose |
|----------|---------|
| **tests.yml** | Unit & Snapshot tests, Code coverage |
| **swiftlint.yml** | Enforces 60+ lint rules |
| **danger.yml** | Automated PR checks (size, coverage, architecture) |
| **periphery.yml** | Dead code detection |
| **docc.yml** | Auto-generates documentation |

---

## Special Notes

- ❌ Do NOT mutate files outside the workspace root without explicit approval
- ❌ Do NOT perform destructive git operations unless requested
- ❌ Do NOT use `DispatchQueue` — use async/await instead
- ❌ Do NOT use `print()` — use `Logger` instead
- ✅ When unsure, ASK the user for guidance
- ✅ Commit only things you modified yourself

---
> Source: [finnchristoffer/RAWG.IO](https://github.com/finnchristoffer/RAWG.IO) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
