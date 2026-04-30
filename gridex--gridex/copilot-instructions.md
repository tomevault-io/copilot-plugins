## gridex

> Native macOS app (Swift, AppKit) combining a powerful database GUI with built-in AI chat.

# Gridex — AI-Native Database IDE for macOS

## Project Overview
Native macOS app (Swift, AppKit) combining a powerful database GUI with built-in AI chat.

## Architecture
Clean Architecture with 5 layers:
- **Core**: Protocols, Models, Enums, Errors, Extensions — zero dependencies
- **Domain**: Use Cases, Repository protocols — depends only on Core
- **Data**: Database adapters (SQLite, PostgreSQL, MySQL, Redis), SwiftData persistence, Keychain — implements Domain
- **Services**: Query Engine, AI (Anthropic/OpenAI/Ollama), SSH Tunnel, Import/Export
- **Presentation**: AppKit views (NSTableView, NSOutlineView, NSSplitView), SwiftUI settings, ViewModels, Coordinators

## Tech Stack
- **Language**: Swift 5.10+
- **UI**: AppKit (primary) + SwiftUI (settings, forms)
- **Persistence**: SwiftData
- **Credentials**: macOS Keychain
- **DB Drivers**: PostgresNIO, MySQLNIO, SQLite3, RediStack
- **SSH**: swift-nio-ssh
- **AI**: Direct API calls to Anthropic/OpenAI, local Ollama

## Key Conventions
- All database adapters conform to `DatabaseAdapter` protocol
- All LLM providers conform to `LLMService` protocol
- Use `actor` for thread-safe services (ConnectionManager, QueryEngine)
- SwiftData `@Model` entities live in Data/Persistence/Models
- Domain models are plain structs (Codable, Sendable)
- Dependency injection via `DependencyContainer`
- Errors centralized in `GridexError` enum

## Folder Structure
- `macos/` — macOS app source code
- `window/` — Windows app (future)
- `Package.swift` — SPM manifest

## Build & Run
```bash
swift build
.build/debug/Gridex
```

---
> Source: [gridex/gridex](https://github.com/gridex/gridex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
