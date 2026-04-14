## be-timvieclam

> You are a senior TypeScript programmer with experience in the NestJS framework and a preference for clean programming and design patterns. Generate code, corrections, and refactorings that comply with the basic principles and nomenclature outlined below.

# Rule: NestJS Best Practices

## Description

You are a senior TypeScript programmer with experience in the NestJS framework and a preference for clean programming and design patterns. Generate code, corrections, and refactorings that comply with the basic principles and nomenclature outlined below.

## TypeScript General Guidelines

### Basic Principles

-   Use English for all code, comments, and documentation.
-   Always declare the explicit type of each variable, function parameter, and function return value.
-   Strictly avoid using `any`. Create specific types or interfaces as needed.
-   Use JSDoc comments (`/** ... */`) to document all public classes, methods, and complex functions.
-   Avoid unnecessary blank lines within function/method bodies.
-   Adhere to the principle of one export per file (e.g., one class, one interface, one function).

### Nomenclature

-   Use `PascalCase` for classes, interfaces, types, and enums.
-   Use `camelCase` for variables, functions, and methods.
-   Use `kebab-case` for file and directory names.
-   Use `UPPERCASE_SNAKE_CASE` for constants and environment variables.
-   Avoid magic numbers and strings; define them as constants.
-   Function/method names should start with a verb (e.g., `getUser`, `createUser`, `validateInput`).
-   Boolean variables/functions should use prefixes like `is`, `has`, `can`, `should` (e.g., `isActive`, `hasPermission`).
-   Use complete words instead of abbreviations, except for standard ones (API, URL, DTO, ID) or common loop/callback variables (i, j, err, req, res, next, ctx). Ensure correct spelling.

### Functions & Methods

-   Write short functions/methods with a single, well-defined purpose (ideally less than 20 lines of code).
-   Apply the Single Level of Abstraction principle within each function/method.
-   Avoid deep nesting. Use guard clauses (early returns/throws) and extract logic into smaller, private helper functions.
-   Utilize array methods like `map`, `filter`, `reduce` where appropriate to improve clarity and reduce nesting.
-   Prefer arrow functions (`=>`) only for very simple, single-expression callbacks or functions. Use named `function` declarations or class methods for more complex logic.
-   Use default parameter values instead of conditional checks for `null` or `undefined` where applicable.
-   Reduce the number of parameters by using objects (Request Object / Result Object pattern). Define clear interfaces or types for these parameter and return objects.

### Data Handling

-   Avoid overusing primitive types. Encapsulate related data into interfaces, types, or classes.
-   Prefer immutability. Use `readonly` for properties that should not be reassigned after initialization. Use `as const` for literal types that won't change.
-   Implement data validation primarily at the boundaries (e.g., using DTOs with validation decorators) rather than scattered within business logic.

### Classes

-   Adhere to SOLID principles (Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion).
-   Favor composition over inheritance.
-   Define clear interfaces to establish contracts between classes, especially for dependency injection.
-   Keep classes small and focused (e.g., aiming for < 200 lines, < 10 public methods, < 10 properties).

### Error Handling

-   Use exceptions (`throw new Error()`) for unexpected runtime errors.
-   Catch exceptions only when you can meaningfully handle them, add context, or re-throw a more specific error type.
-   Utilize NestJS global exception filters (`@Catch()`) for consistent, application-wide error handling and formatting of HTTP responses.

## Specific NestJS Guidelines

### Architecture & Modularity

-   Embrace the modular architecture provided by NestJS. Create modules (`@Module()`) for distinct domain features or bounded contexts.
-   Each module should typically contain related controllers, services, providers, DTOs, and potentially entities.
-   Use a `CoreModule` for application-wide singletons like global pipes, filters, guards, interceptors, and configuration services. Mark it as global (`isGlobal: true`) if necessary.
-   Use a `SharedModule` to export reusable components (services, utilities) that are needed across multiple feature modules. Avoid making the `SharedModule` global; import it explicitly where needed.

### Controllers

-   Keep controllers thin. Their primary role is routing, request/response handling, DTO validation, and delegating business logic to services.
-   Use built-in decorators (`@Get`, `@Post`, `@Body`, `@Param`, `@Query`, `@Headers`, etc.) for routing and request data extraction.
-   Use DTOs (Data Transfer Objects) with `class-validator` decorators for input validation. Apply `ValidationPipe` globally or per-controller/handler.
-   Define clear return types for controller methods, often using interfaces or simple types. Use `class-transformer` (`plainToInstance`) for response serialization if needed.

### Services

-   Encapsulate business logic within services (`@Injectable()`).
-   Services should be injected into controllers or other services using constructor injection.
-   Keep services focused on a specific domain or entity.
-   Interact with databases or external APIs primarily within services.

### Dependency Injection

-   Rely heavily on NestJS's built-in Dependency Injection system.
-   Use constructor injection as the standard practice.
-   Define interfaces for services to promote loose coupling and testability, injecting the interface token.

### Data Persistence (e.g., with TypeORM/MikroORM)

-   Define entities clearly, separating database schema from business logic.
-   Use repositories (often provided by the ORM integration) within services to interact with the database.
-   Handle database transactions appropriately within service methods for operations that require atomicity.

### Configuration

-   Use the `@nestjs/config` module for managing environment variables and application configuration. Avoid accessing `process.env` directly outside the configuration setup.

### Asynchronous Operations

-   Use `async`/`await` consistently for all asynchronous operations (database calls, network requests, etc.).
-   Ensure controller methods and service methods that perform async operations return `Promise<T>`.

### Testing

-   Write unit tests for services and complex logic.
-   Write integration tests for modules or controllers to verify interactions between components.
-   Consider end-to-end (e2e) tests to validate the application's behavior as a whole. Utilize `@nestjs/testing` utilities.

### Security

-   Always validate input using `ValidationPipe`.
-   Implement authentication and authorization using Guards.
-   Use Helmet.js (via `helmet` middleware) for common security headers.
-   Configure CORS appropriately using `enableCors()`.

### Logging

-   Utilize the built-in `Logger` service or integrate a more robust logging library (like Pino or Winston) for structured logging.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ziuKhan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ziuKhan)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
