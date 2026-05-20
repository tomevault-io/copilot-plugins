## agentsdk-swift

> swift test --filter AgentSDK_SwiftTests/testSpecificFunction

# AgentSDK-Swift Guidelines

## Build & Test Commands
```bash
# Build the project
swift build

# Run all tests
swift test

# Run a specific test
swift test --filter AgentSDK_SwiftTests/testSpecificFunction

# Run a group of tests
swift test --filter AgentSDK_SwiftTests

# Run example app (requires API key)
export OPENAI_API_KEY=your_api_key_here
swift run SimpleApp
```

## Test Guidelines
- Use `@Test` annotation to mark test functions
- Use `#expect` assertions for validation
- Follow AAA pattern (Arrange, Act, Assert)
- Test each class & function independently
- Use descriptive test names starting with "test"
- Group tests with `// MARK: - Category` comments
- Create separate test files for complex classes
- Test public interfaces rather than implementation details
- Use `@testable import` to access internal members
- Handle platform-specific imports in test files

## Code Style Guidelines

### Formatting & Structure
- Use 4-space indentation
- PascalCase for types (classes, structs, enums)
- camelCase for functions, variables, parameters
- Explicit visibility modifiers (public, internal, private)
- Triple-slash (///) for documentation comments

### Types & Error Handling
- Protocol-oriented design with clear interfaces
- Use generics for type safety
- Structured error types with nested enums
- Consistent do-catch blocks and error propagation
- Include descriptive error messages

### Swift Practices
- Minimal imports (Foundation first)
- Use URLSession for networking (not AsyncHTTPClient)
- Handle platform differences with conditional imports (`#if canImport(FoundationNetworking)`)
- Async/await for asynchronous operations
- Immutable data structures where possible
- Dependency injection via initializers
- Comprehensive unit tests for core functionality

---
> Source: [fumito-ito/AgentSDK-Swift](https://github.com/fumito-ito/AgentSDK-Swift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
