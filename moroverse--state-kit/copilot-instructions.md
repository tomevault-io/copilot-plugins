## state-kit

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test Commands

```bash
swift build          # Build the library
swift test           # Run all tests
swift test --filter StateKitTests.ListStoreTests          # Run a single test suite
swift test --filter StateKitTests.ListStoreTests/testName # Run a single test
```

Formatting: `swiftformat .` (configured via `.swiftformat`, installed via mise — see `.mise.toml`)

## Architecture

StateKit is a Swift library for managing async list/detail loading state, built on `@Observable` and Swift concurrency. Targets iOS 17+, macOS 14+, macCatalyst 17+. Swift tools version 6.2.

### Composable Decorator Pattern

The core design is a **decorator chain**. Start with a base `ListStore` and wrap it with decorators to add features:

```swift
ListStore(loader: api.fetch, queryFactory: { .default })
    .searchable(queryBuilder: { term in Query(term: term) })  // → SearchableListStore
    .paginated()                                               // → PaginatedListStore
    .selectable()                                               // → SelectableListStore
```

Each decorator conforms to `ListStateProviding` and delegates to its wrapped store. The fluent API is powered by `ListStoreComposable`, an internal protocol giving decorators access to the root `ListStore`.

### Key Types

- **`ListStore<Model, Query, Failure>`** — Core `@Observable` store for async collection loading. Manages state machine (`ListLoadingState`), caching, and cancellation. Delegates to `LoadingEngine`.
- **`DetailStore<Model, Query, Failure>`** — Single-item counterpart using `LoadingState` (no pagination). Uses `DataLoaderActor` for query-based caching.
- **Decorator stores** — `SearchableListStore` (debounced text search via `SearchEngine`), `PaginatedListStore` (load-more via `PaginationEngine`), `SelectableListStore` (selection tracking).
- **Paginators** — `CursorPaginator` (generic cursor-based, supports composite cursors), `OffsetPaginator` (offset-based). Both are actors with caching and local update support via `Difference`.
- **`Paginated<Item>`** — Wraps items array + optional `loadMore` closure. Conforms to `RandomAccessCollection`.

### Protocol Hierarchy

Protocols are orthogonal traits designed for free composition via conditional conformances:

- `ListStateProviding` — Base: `state`, `element(at:)`, `load(forceReload:)`
- `SearchableListProviding` — Adds `search(_:)`, `cancelSearch()`
- `PaginatedListProviding` — Adds `loadMore()`
- `SelectableListProviding` — Adds `selection`, `select(_:)`

### State Machine

`ListLoadingState` is an indirect enum: `idle → inProgress → loaded/empty/error`. The `inProgress` and `error` cases carry `previousState` for UI continuity. `LoadMoreState` is nested within `loaded`.

### Internal Engines

Logic is extracted into reusable engine types to avoid duplication across stores:
- `LoadingEngine` — Load, cache, task management, state transitions
- `SearchEngine` — Query string coordination with `Debounce` actor
- `PaginationEngine` — Pagination state and load-more lifecycle

## Code Style

- Swift 6.0 concurrency strict; all stores are `@MainActor`
- 4-space indentation, LF line endings
- Wrap arguments/collections before first
- SwiftLint: max line length 160, nesting warning at depth 2
- Tests use Swift Testing framework (not XCTest) with `TestKit` (TestClock, AsyncSpy, memory leak tracking)
- Dependencies: `swift-clocks` (pointfreeco) for testable async timing, `test-kit` (Moroverse) for test utilities

# Important

1. Prioritize substance over compliments. Never soften criticism. If an idea has holes, say so directly-"This won't scale because X" is better than "Have you considered...". Challenge assumptions. Point out errors. Useful feedback matters more than comfortable feedback.

2. Do not add Co-Authored-By: Claude **** <noreply@anthropic.com>" in commit messages

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Moroverse) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
