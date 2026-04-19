## catch-the-fly-back

> - Use English for all code and documentation.

### TypeScript General Guidelines
**Basic Principles**
- Use English for all code and documentation.
- Explicitly declare types for all variables and functions.
- Avoid `any`; create types as needed.
- Document public classes and methods with JSDoc.
- One export per file; no blank lines within functions.
**Nomenclature**
- PascalCase: classes, camelCase: variables, functions, methods, kebab-case: files and directories.
- UPPERCASE for environment variables; define constants for "magic numbers."
- Verb-based boolean names (e.g., `isLoading`, `canDelete`).
- Avoid abbreviations, except for standard ones (API, URL) and specific short ones (i, j for loops, err for errors).
**Functions**
- Short, single-purpose functions (under 20 instructions).
- Use `isX`, `hasX` for boolean-returning functions; `executeX`, `saveX` for void functions.
- Avoid block nesting; prefer early returns, utility functions, higher-order functions.
- Use arrow functions for short, simple functions.
- Reduce parameters by passing/returning objects; use default values for optional parameters.
**Data**
- Prefer composite types over primitives; validate within classes.
- Favor immutability; use `readonly` for stable data and `as const` for literals.
**Classes**
- Follow SOLID principles, preferring composition.
- Define interfaces for contracts, create single-purpose classes (under 200 lines, 10 methods, 10 properties).
**Exceptions**
- Use exceptions for unexpected errors; handle expected errors with context or a global handler.
**Testing**
- Use Arrange-Act-Assert convention; name test variables clearly.
- Write unit tests for each public function, and acceptance tests for each module.
- Use test doubles for dependencies (except for inexpensive third-party ones).

---
### NestJS-Specific Guidelines
**Architecture**
- Use modular architecture:
 - One module per domain/route.
 - One controller per route; add secondary controllers as needed.
 - Models folder for data types and validated DTOs.
 - Services module with one service per entity, using MikroORM for persistence.
 - Core module for global filters, middlewares, guards, and interceptors.
 - Shared module for reusable services and utilities.
**Testing**
- Use Jest for tests.
- Write tests for each controller and service.
- Add end-to-end tests for each API module and a basic smoke test method (`/admin/test`) in each controller.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/taekyun0219) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
