## poem-of-the-day

> - **Use `async/await`** for asynchronous operations instead of completion handlers

# Swift Development Patterns & Best Practices

## Modern Swift Patterns
- **Use `async/await`** for asynchronous operations instead of completion handlers
- **Use `@MainActor`** for UI updates and main thread operations
- **Use `actor`** for thread-safe shared state
- **Use `Task`** for structured concurrency
- **Use `defer`** for cleanup operations

## Error Handling
- Define custom error types conforming to `Error` protocol
- Use `Result<Success, Failure>` for operations that can fail
- Prefer `throws` for functions that can fail
- Use `do-catch` blocks for error handling
- Create meaningful error messages for debugging

## SwiftUI Best Practices
- Use `@StateObject` for creating observable objects
- Use `@ObservedObject` for passing observable objects
- Use `@State` for local view state
- Use `@Binding` for two-way data binding
- Prefer `ViewBuilder` for complex view composition
- Use `@Environment` for shared configuration

## MVVM Implementation
- ViewModels should be `ObservableObject` classes
- Use `@Published` for properties that trigger UI updates
- Keep business logic in ViewModels, not Views
- Use dependency injection for services
- Make ViewModels testable by accepting protocols

## Dependency Injection
- Use protocols for service abstractions
- Inject dependencies through initializers
- Use a dependency container for complex apps
- Mock dependencies for testing
- Prefer composition over inheritance

## Code Organization
- Group related functionality in extensions
- Use `// MARK: -` comments for organization
- Keep files focused on single responsibilities
- Use descriptive names for functions and variables
- Add documentation comments for public APIs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ssk42) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
