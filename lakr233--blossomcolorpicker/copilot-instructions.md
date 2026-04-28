## blossomcolorpicker

> The user may be using voice input. If words seem misspelled, unclear, or the wrong word appears to be used, please ask for clarification rather than guessing.

    # Swift Code Style Guidelines

## Voice Input Notice

The user may be using voice input. If words seem misspelled, unclear, or the wrong word appears to be used, please ask for clarification rather than guessing.

## Available Skills

Use `/skill-name` to invoke these skills:

### Swift & SwiftUI
- `/swiftui-expert-skill` - Write, review, or improve SwiftUI code with best practices
- `/swiftui-liquid-glass` - iOS 26+ Liquid Glass API implementation
- `/swiftui-performance-audit` - Audit and improve SwiftUI runtime performance
- `/swiftui-ui-patterns` - SwiftUI view patterns and component guidance
- `/swiftui-view-refactor` - Refactor SwiftUI views for consistent structure
- `/swift-concurrency` - Swift Concurrency best practices (async/await, actors)
- `/swift-concurrency-expert` - Swift Concurrency review and remediation for Swift 6.2+
- `/core-data-expert` - Core Data guidance for iOS/macOS

### iOS Development
- `/ios-debugger-agent` - Build, run, and debug iOS projects on simulator
- `/macos-spm-app-packaging` - SwiftPM-based macOS app packaging

### Git & GitHub
- `/github` - Interact with GitHub using `gh` CLI
- `/gh-issue-fix-flow` - End-to-end GitHub issue fix workflow
- `/app-store-changelog` - Generate App Store release notes from git history

## Core Style

- **Indentation**: 4 spaces
- **Braces**: Opening brace on same line
- **Spacing**: Single space around operators and commas
- **Naming**: PascalCase for types, camelCase for properties/methods

## File Organization

- Logical directory grouping
- PascalCase files for types, `+` for extensions
- Modular design with extensions

## Modern Swift Features

- **@Observable macro**: Replace `ObservableObject`/`@Published`
- **Swift concurrency**: `async/await`, `Task`, `actor`, `@MainActor`
- **Result builders**: Declarative APIs
- **Property wrappers**: Use line breaks for long declarations
- **Opaque types**: `some` for protocol returns

## Code Structure

- Early returns to reduce nesting
- Guard statements for optional unwrapping
- Single responsibility per type/extension
- Value types over reference types

## Error Handling

- `Result` enum for typed errors
- `throws`/`try` for propagation
- Optional chaining with `guard let`/`if let`
- Typed error definitions

## Architecture

- Protocol-oriented design
- Dependency injection over singletons
- Composition over inheritance
- Factory/Repository patterns

## Debug Assertions

- Use `assert()` for development-time invariant checking
- Use `assertionFailure()` for unreachable code paths
- Assertions removed in release builds for performance
- Precondition checking with `precondition()` for fatal errors

## Memory Management

- `weak` references for cycles
- `unowned` when guaranteed non-nil
- Capture lists in closures
- `deinit` for cleanup

---
> Source: [Lakr233/BlossomColorPicker](https://github.com/Lakr233/BlossomColorPicker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
