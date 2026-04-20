## headchef

> - Use English for all code and documentation.

# Coding Standards

# Base Rules

## Basic Principles

- Use English for all code and documentation.
- Always declare the type of each variable and function (parameters and return value).
  - Avoid using any.
  - Create necessary types.
- Don't leave blank lines within a function.
- One export per file.

## Nomenclature

- Use PascalCase for classes.
- Use camelCase for variables, functions, and methods.
- Use underscores_case for file and directory names.
- Use UPPERCASE for environment variables.
  - Avoid magic numbers and define constants.
- Start each function with a verb.
- Use verbs for boolean variables. Example: isLoading, hasError, canDelete, etc.
- Use complete words instead of abbreviations and correct spelling.
  - Except for standard abbreviations like API, URL, etc.
  - Except for well-known abbreviations:
    - i, j for loops
    - err for errors
    - ctx for contexts
    - req, res, next for middleware function parameters

## Functions

- Write short functions with a single purpose. Less than 20 instructions.
- Name functions with a verb and something else.
  - If it returns a boolean, use isX or hasX, canX, etc.
  - If it doesn't return anything, use executeX or saveX, etc.
- Avoid nesting blocks by:
  - Early checks and returns.
  - Extraction to utility functions.
- Use higher-order functions (map, filter, reduce, etc.) to avoid function nesting.
  - Use arrow functions for simple functions (less than 3 instructions).
  - Use named functions for non-simple functions.
- Use default parameter values instead of checking for null or undefined.
- Reduce function parameters using RO-RO (Receive an Object, Return an Object).
  - Use an object to pass multiple parameters.
  - Use an object to return results.
  - Declare necessary types for input arguments and output.
- Use a single level of abstraction.

## Data

- Don't abuse primitive types and encapsulate data in composite types.
- Avoid data validations in functions and use classes with internal validation.
- Prefer immutability for data.
  - Use readonly for data that doesn't change.
  - Use as const for literals that don't change.

## Classes

- Follow SOLID principles.
- Prefer composition over inheritance.
- Declare interfaces to define contracts.
- Write small classes with a single purpose.
  - Less than 200 instructions.
  - Less than 10 public methods.
  - Less than 10 properties.

## Exceptions

- Use exceptions to handle errors you don't expect.
- If you catch an exception, it should be to:
  - Fix an expected problem.
  - Add context.
  - Otherwise, use a global handler.

## Testing

- Follow the Arrange-Act-Assert convention for tests.
- Name test variables clearly.
  - Follow the convention: inputX, mockX, actualX, expectedX, etc.
- Write unit tests for each public function.
  - Use test doubles to simulate dependencies.
    - Except for third-party dependencies that are not expensive to execute.
- Write acceptance tests for each module.
  - Follow the Given-When-Then convention.

## SOLID Principles

### Single Responsibility Principle (S)

Each class or function should have only one reason to change. Separate concerns into different modules.

### Open/Closed Principle (O)

Software entities should be open for extension but closed for modification. Use abstractions and interfaces.

### Liskov Substitution Principle (L)

Objects of a superclass should be replaceable with objects of subclasses without breaking the application.

### Interface Segregation Principle (I)

Many client-specific interfaces are better than one general-purpose interface. Don't force implementations to depend on methods they don't use.

### Dependency Inversion Principle (D)

Depend on abstractions, not concretions. High-level modules should not depend on low-level modules.

## Design Patterns

### Repository Pattern

Abstract data access logic behind interfaces. Repositories handle data retrieval and persistence.

### Factory Pattern

Create objects without exposing creation logic. Use factories when object creation is complex.

### Strategy Pattern

Define a family of algorithms, encapsulate each one, and make them interchangeable.

### Observer Pattern

Define a one-to-many dependency between objects so that when one object changes state, all its dependents are notified.

## Security Principles

- Never hardcode secrets or API keys.
- Use environment variables for configuration.
- Validate all user input.
- Sanitize data before display.
- Use HTTPS for all network requests.
- Implement proper authentication and authorization.
- Keep dependencies updated.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/srcnysf) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
