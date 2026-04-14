## backend-world-wide

> These guidelines establish a comprehensive framework for building robust, maintainable TypeScript applications with NestJS. They are designed to promote consistency, readability, and long-term sustainability across complex enterprise applications.

# Enhanced TypeScript & NestJS Development Guidelines

These guidelines establish a comprehensive framework for building robust, maintainable TypeScript applications with NestJS. They are designed to promote consistency, readability, and long-term sustainability across complex enterprise applications.

## TypeScript Core Principles

### Fundamental Standards

- Use precise, meaningful English for all code and documentation.
- Employ strong typing throughout the codebase:
  - Explicitly declare types for all variables, parameters, and return values.
  - Avoid `any` type unless absolutely necessary; prefer `unknown` when type is truly undetermined.
  - Create domain-specific types and interfaces to enhance code semantics.
  - Leverage TypeScript's discriminated unions and generics for complex type scenarios.
- Document public APIs thoroughly with JSDoc:
  - Include detailed descriptions, parameter explanations, return value documentation, and examples.Pr
  - Document expected exceptions and edge cases.
- Maintain clean code structure:
  - Eliminate unnecessary blank lines within functions.
  - Limit each file to a single primary export.
  - Keep files concise and focused on a single responsibility.
- Utilize TypeScript's latest features when they improve code quality:
  - Optional chaining (`?.`)
  - Nullish coalescing (`??`)
  - Non-null assertion operator (`!`) only when absolutely necessary
  - Template literal types for complex string manipulation

### Naming Conventions

- Follow consistent casing patterns:
  - `PascalCase` for classes, interfaces, types, enums, and decorators
  - `camelCase` for variables, functions, methods, and properties
  - `kebab-case` for file and directory names
  - `UPPER_SNAKE_CASE` for constants and environment variables
- Create semantic and self-descriptive names:
  - Use domain-specific terminology consistently
  - Begin function names with clear action verbs
  - Prefix boolean variables with verbs: `isEnabled`, `hasPermission`, `canModify`
  - Use complete, correctly spelled words over abbreviations
  - Acceptable abbreviations include:
    - Standard technical terms (API, URL, JWT, UUID)
    - Loop counters (`i`, `j`, `k`)
    - Error handling (`err`, `error`)
    - Context objects (`ctx`, `context`)
    - HTTP paradigms (`req`, `res`, `next`)
- Define domain-specific lexicon for consistent terminology across the application

### Function Design

- Create highly cohesive functions with single responsibility:
  - Maximum 20 logical operations per function
  - Clear input/output contracts
  - Consistent error handling strategy
- Follow semantic naming patterns:
  - Action verbs + object for most functions: `calculateTotal`, `validateInput`
  - Predicates for boolean returns: `isValid`, `hasAccess`, `canProceed`
  - Event handlers: `onSubmit`, `handleChange`
  - State mutations: `updateState`, `saveChanges`, `persistData`
- Optimize for readability and maintainability:
  - Implement early returns for preconditions
  - Extract complex logic to well-named helper functions
  - Limit nesting depth to 2-3 levels
  - Utilize functional programming patterns where appropriate
- Enforce parameter discipline:
  - Use destructured objects for functions with >2 parameters (Receive-object, Return-object pattern)
  - Provide meaningful default values
  - Define clear interface types for complex parameters
  - Implement parameter validation at function boundaries
- Maintain single abstraction level within each function

### Data Management

- Employ rich domain models over primitive obsession:
  - Encapsulate related data in purpose-built types
  - Utilize value objects for concepts with intrinsic rules (Email, Currency, PhoneNumber)
  - Implement domain entities with identity and behavior
- Enforce data integrity:
  - Implement validation at domain boundaries
  - Use class-validator for DTO validation
  - Create factory methods for complex object construction
  - Consider runtime type checking for external data
- Prioritize immutability:
  - Use `readonly` for properties that should not change
  - Apply `as const` for immutable literals and arrays
  - Implement proper copy mechanisms for state updates
  - Consider Immer or similar libraries for complex state management
- Design clear data flow:
  - Distinguish between commands and queries
  - Establish consistent data transformation patterns
  - Define clear boundaries between layers

### Class Architecture

- Adhere to SOLID principles:
  - **S**ingle Responsibility: Each class should have only one reason to change
  - **O**pen/Closed: Open for extension, closed for modification
  - **L**iskov Substitution: Subtypes must be substitutable for their base types
  - **I**nterface Segregation: Many client-specific interfaces are better than one general-purpose interface
  - **D**ependency Inversion: Depend on abstractions, not concretions
- Implement compositional design:
  - Favor composition over inheritance
  - Use mixins or higher-order functions when appropriate
  - Apply the strategy pattern for variant behaviors
  - Utilize the decorator pattern for cross-cutting concerns
- Maintain manageable class sizes:
  - Maximum ~200 logical operations per class
  - No more than 10 public methods
  - Limited property count (≤10)
  - Consider splitting when responsibilities expand
- Define clear contracts:
  - Use interfaces to define public APIs
  - Implement abstract classes for shared behavior
  - Apply generics for reusable components
  - Consider branded types for type safety

### Error Handling

- Implement a strategic exception management system:
  - Create a hierarchy of application-specific exceptions
  - Use exceptions for exceptional conditions, not control flow
  - Include context information in exception messages
  - Create factory methods for common exceptions
- Follow consistent error patterns:
  - Catch exceptions to add context or handle recoverable errors
  - Propagate unrecoverable exceptions to global handlers
  - Transform technical exceptions to domain-specific ones at boundaries
  - Log detailed error information while presenting user-friendly messages
- Design for resilience:
  - Implement retry mechanisms for transient failures
  - Use circuit breakers for dependent services
  - Consider fallback strategies for critical operations
  - Validate external data at system boundaries

### Testing Strategy

- Implement comprehensive test coverage:
  - Unit tests for all public functions
  - Integration tests for module interactions
  - End-to-end tests for critical user flows
  - Performance tests for bottleneck operations
- Follow testing best practices:
  - Arrange-Act-Assert (AAA) pattern for unit tests
  - Given-When-Then structure for behavior-driven tests
  - Property-based testing for algorithmic operations
  - Snapshot testing for UI components
- Use clear test naming conventions:
  - Descriptive test function names: `should_returnFilteredResults_whenGivenValidParams`
  - Consistent variable naming: `inputData`, `mockService`, `actualResult`, `expectedOutput`
- Employ effective test doubles:
  - Mocks for verifying interactions
  - Stubs for providing test data
  - Fakes for simulating complex dependencies
  - Spies for observing behavior
  - In-memory implementations for data stores

## NestJS Architecture

### Modular Design

- Implement domain-driven modular architecture:
  - Organize by business capability or bounded context
  - Maintain clear module boundaries
  - Define explicit public APIs between modules
  - Minimize cross-module dependencies
- Structure each module consistently:
  - One primary module class
  - Feature-focused controllers
  - Domain-specific services
  - Well-defined data models
  - Clear separation of concerns
- Design modules for testability and reusability:
  - Export only necessary components
  - Use feature modules for specialized functionality
  - Implement shared modules for cross-cutting concerns
  - Consider dynamic modules for configurable components

### API Design

- Create RESTful APIs following best practices:
  - Implement resource-oriented endpoints
  - Use appropriate HTTP methods for operations
  - Return meaningful HTTP status codes
  - Design URLs with consistent patterns
  - Include pagination, filtering, and sorting capabilities
- Enforce strict API validation:
  - Define comprehensive DTOs with class-validator
  - Implement transformation with class-transformer
  - Create specialized pipes for complex validation
  - Document validation rules in OpenAPI/Swagger
- Implement standardized response structures:
  - Consistent success response format
  - Uniform error response schema
  - Include metadata for collections
  - Support partial responses for large resources
- Design for evolution:
  - Implement API versioning strategy
  - Consider hypermedia controls
  - Document deprecation policies
  - Plan for backward compatibility

### Data Persistence

- Implement effective data access patterns:
  - Use the repository pattern for data access
  - Implement unit of work for transaction management
  - Consider CQRS for complex applications
  - Optimize for both read and write operations
- Leverage MikroORM effectively:
  - Define clean entity models
  - Use migrations for database schema evolution
  - Implement appropriate indexing strategies
  - Configure optimistic locking for concurrent operations
- Design for performance and scalability:
  - Implement efficient querying patterns
  - Use eager/lazy loading appropriately
  - Consider caching for frequently accessed data
  - Implement pagination for large datasets
- Ensure data integrity:
  - Use transactions for related operations
  - Implement database constraints
  - Design for eventual consistency where appropriate
  - Consider audit logging for sensitive operations

### Service Implementation

- Create focused service components:
  - One service per primary entity or use case
  - Clear separation between command and query services
  - Application services for orchestration
  - Domain services for business logic
- Implement dependency injection effectively:
  - Use constructor injection for required dependencies
  - Consider property injection for optional dependencies
  - Leverage custom providers for complex scenarios
  - Use appropriate injection scopes (default, request, transient)
- Design for maintainability:
  - Create stateless services when possible
  - Implement clear lifecycles for stateful services
  - Use decorators for cross-cutting concerns
  - Consider middleware for request processing

### Core Infrastructure

- Implement robust application infrastructure:
  - Global exception filters with structured error responses
  - Request/response logging middleware
  - Authentication and authorization guards
  - Performance monitoring interceptors
- Design for observability:
  - Structured logging with correlation IDs
  - Metrics collection for key operations
  - Distributed tracing for request flows
  - Health check endpoints
- Implement security best practices:
  - Input validation at all entry points
  - Output encoding for all responses
  - Rate limiting for public endpoints
  - CSRF protection for browser clients

### Advanced Testing

- Implement comprehensive test coverage:
  - Unit tests with isolated dependencies
  - Integration tests with test databases
  - End-to-end tests with running services
  - Load tests for performance-critical paths
- Use testing utilities effectively:
  - NestJS testing module for dependency injection
  - Test fixtures for common test data
  - Factory functions for test entities
  - Custom test utilities for repeated patterns
- Implement specialized testing practices:
  - Contract tests for API boundaries
  - Consumer-driven contract tests for service integration
  - Chaos testing for resilience
  - Security tests for vulnerability detection
- Automate testing process:
  - Continuous integration with test execution
  - Code coverage reporting
  - Mutation testing for test quality
  - Visual regression testing for UI components

## Development Workflow

### Code Organization

- Maintain consistent project structure:
  - Feature-oriented folders
  - Clear separation between API, domain, and infrastructure
  - Standardized file naming conventions
  - Logical grouping of related components
- Manage dependencies effectively:
  - Regular dependency updates
  - Dependency pinning for stability
  - Sharing of common dependencies via shared modules
  - Careful evaluation of third-party libraries

### Quality Assurance

- Implement comprehensive code quality tools:
  - ESLint with custom rule sets
  - Prettier for code formatting
  - Husky for pre-commit hooks
  - SonarQube or similar for advanced static analysis
- Enforce code review processes:
  - Clear review guidelines
  - Automated checks before human review
  - Focus on design and maintainability
  - Knowledge sharing during reviews

### Documentation

- Create comprehensive documentation:
  - API documentation with OpenAPI/Swagger
  - Technical documentation for architecture
  - Inline code documentation for complex logic
  - Decision records for significant architectural choices
- Maintain living documentation:
  - Generated from code where possible
  - Regular review and updates
  - Version control for documentation
  - Accessibility for all team members

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/codeslw) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
