## xmlkit

> XMLKit is a pure-Swift, zero-dependency XML parser and formatter.

# XMLKit — Development Guide

## Project Overview

XMLKit is a pure-Swift, zero-dependency XML parser and formatter.

## Build & Test Commands

```bash
swift build          # Build the library
swift test           # Run the test suite
```

## Lint & Format

```bash
swiftlint lint --strict                              # Lint
swift format lint --strict --recursive Sources/ Tests/  # Check formatting
swift format --recursive Sources/ Tests/               # Auto-format
```

## Project Structure

```
Sources/XMLKit/       — Library source code
Tests/XMLKitTests/    — Tests (uses Swift Testing framework)
Tests/XMLKitTests/Fixtures/ — XML test fixture files
API.md                — Public API reference (keep in sync with source)
```

## Conventions

- **Swift 6.0** with strict concurrency enabled via swift-tools-version 6.0
- **Swift Testing** framework for tests (`import Testing`, `@Test` functions) — not XCTest
- **No external dependencies** — the library must remain pure Swift
- **4-space indentation** everywhere
- **Line length limit**: 120 chars (warning), 150 chars (error)
- Test fixtures go in `Tests/XMLKitTests/Fixtures/` and are accessed via `Bundle.module`
- Follow Swift API Design Guidelines: clear names, prefer methods over free functions, use Swift naming conventions (camelCase for functions/properties, PascalCase for types)

## API Documentation

`API.md` is the public API reference for XMLKit. **When adding, removing, or changing any public API** (types, methods, properties, initializers, enum cases), update `API.md` to reflect the change.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/steelbrain) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
