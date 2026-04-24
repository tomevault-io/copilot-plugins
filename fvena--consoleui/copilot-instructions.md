## consoleui

> TypeScript and Code Quality Guidelines

# TypeScript and Code Quality Guidelines

These rules provide guidance on TypeScript usage, code quality, and architectural patterns for this library project.

## TypeScript Development

- Maintain strict typing throughout the codebase
- Export public APIs only from index.ts files
- Keep utility functions in their own modules
- Use TypeScript interface patterns for public APIs
- Ensure all exported functions and types have proper JSDoc comments
- Maintain type definitions when modifying code
- Avoid using `any` - use `unknown` when type is not known
- Leverage the type system to prevent errors at compile time

## TypeScript Typing

- Implement interfaces to define public contracts
- Use types for unions, intersections, and utility types
- Leverage generics to create reusable typed components
- Implement type guards for runtime type checking
- Use explicit return types for public functions
- Apply literal types for restricted values
- Implement enums for fixed sets of related values
- Use readonly modifiers for immutable properties

## Advanced Typing

- Use mapped types for type transformations
- Implement conditional types for type-based logic
- Apply type inference correctly
- Use TypeScript utility types (Partial, Required, Pick, etc.)
- Implement custom type guards with predicates
- Use type assertions only when absolutely necessary
- Define recursive types for nested structures
- Apply type templating for generic components

## Software Architecture

- Follow the Single Responsibility Principle (SRP) for all classes and functions
- Implement modular architecture with clear boundaries between components
- Organize code following separation of concerns
- Maintain a layered architecture when appropriate
- Separate domain logic from infrastructure
- Avoid circular dependencies between modules
- Implement dependency inversion pattern when appropriate
- Apply SOLID principles in design

## Design Patterns

- Use factory patterns for complex object creation
- Implement Builder pattern for objects with multiple configurations
- Apply Singleton pattern only when strictly necessary
- Use Observer patterns for event handling and notifications
- Implement Strategy patterns for interchangeable behaviors
- Use decorators to extend functionality without modifying existing code
- Apply composition over inheritance
- Implement chains of responsibility for sequential processing

## Naming Conventions

- Use PascalCase for class, interface, and type names
- Apply camelCase for variable, property, and function names
- Use UPPER_SNAKE_CASE for constants and enum values
- Prefix 'I' for interfaces only when necessary for clarity
- Prefix 'T' for generic types when it helps clarify
- Use descriptive names and avoid non-standard abbreviations
- Apply meaningful suffixes: `Service`, `Repository`, `Manager`, `Factory`
- Maintain consistency with existing project conventions

## Code Quality

- Maintain ESLint and Prettier configuration
- Don't disable or ignore linting rules
- Follow the established code formatting style
- Keep code modular and focused
- Implement proper error handling
- Use async/await pattern for asynchronous operations
- Write self-documenting code with clear variable/function names
- Keep functions focused and reasonably sized

## Error Handling

- Create custom error hierarchy when appropriate
- Use discriminated unions for typed errors
- Implement Result pattern for operations that may fail
- Propagate errors consistently (throw vs return)
- Document error behavior in public interfaces
- Implement centralized error handling when appropriate
- Use exceptions for exceptional errors, not control flow
- Provide descriptive and actionable error messages

## Domain Modeling

- Create domain models that represent business concepts
- Implement value objects for immutable concepts
- Use entities for objects with unique identity
- Define aggregates to maintain domain consistency
- Implement DTOs for data transfer between layers
- Keep domain models independent from infrastructure
- Apply domain invariants through validations
- Use factory methods to create complex domain objects

## State Management

- Minimize global mutable state
- Design for immutability when possible
- Implement patterns for immutable state updates
- Centralize state management when appropriate
- Clearly define state flows and transitions
- Encapsulate state changes within their respective components
- Implement state validation to maintain consistency
- Use state machine patterns for complex flows

## Composition and Reusability

- Favor composition over inheritance
- Create small, composable utility functions
- Implement hooks and HOCs (in React) for reusability
- Use mixins with TypeScript when appropriate
- Apply DRY principle without sacrificing clarity
- Extract common logic to shared services
- Implement reusable abstractions for recurring patterns
- Create typed utilities for common operations

## Functional Programming

- Prefer pure functions without side effects when possible
- Treat data as immutable
- Use function composition for complex operations
- Implement higher-order functions where appropriate
- Avoid shared mutable state
- Separate data and behavior
- Use methods like map, filter, reduce instead of imperative loops
- Use optional/nullable types instead of undefined checks

## Dos

- Apply SOLID principles in design
- Create abstractions with clear purpose
- Use the type system to prevent errors
- Favor consistency over personal preferences
- Design intuitive APIs that are hard to use incorrectly
- Implement tests that validate type contracts
- Maintain interface segregation when appropriate
- Design for extensibility at strategic points

## Don'ts

- Don't create deep inheritance hierarchies
- Don't use numeric enums without explicit string values
- Don't add getters/setters without purpose
- Don't overload functionality in monolithic classes
- Don't use namespace when ES modules are appropriate
- Don't abuse complex types that hinder readability
- Don't sacrifice maintainability for premature optimizations
- Don't implement patterns just for the sake of it without clear need

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fvena) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
