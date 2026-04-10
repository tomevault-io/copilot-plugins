## revurb

> Read @docs/contributing.md


Read @docs/contributing.md

## IMPORTANT

- All files should end with a single new line
- Try to keep things in one function unless composable or reusable
- DO NOT do unnecessary destructuring of variables
- DO NOT use `else` statements unless necessary
- DO NOT use `try`/`catch` if it can be avoided
- AVOID `try`/`catch` where possible
- AVOID `else` statements
- AVOID using `any` type
- AVOID `let` statements
- PREFER single word variable names where possible
- Use as many Bun APIs as possible like Bun.file() and Bun.env (see https://bun.com/reference/bun)
- ALWAYS USE PARALLEL TOOLS WHEN APPLICABLE.

## Coding Guidelines

We adopt a structured, class-based architecture with strict TypeScript practices, inspired by robust libraries like [Laravel Echo](https://github.com/laravel/echo).

### Architecture
- **Class-Based Core**: Use classes for main logic and services (e.g., `Manager`, `Connector`, `Service`).
- **Contracts & Abstractions**: Use `abstract class` or `interface` to define contracts for components that may have multiple implementations.
- **Dependency Injection**: Pass configuration and dependencies via the constructor. Avoid hardcoding dependencies within classes.
- **Singleton/Global State**: For shared resources (like connections), use module-level singletons or managed state carefully.

### TypeScript Practices
- **Strict Mode**: Always use TypeScript's strict mode.
- **Explicit Return Types**: Explicitly define return types for all methods and functions.
- **Generics**: Use Generics (`<T>`) to create flexible, reusable components and strongly-typed hooks.
- **Type Definitions**: Define shared types and interfaces in dedicated `types.ts` files.
- **Avoid `any`**: Use `unknown` if the type is truly not known, but prefer specific types or generics.

### Code Style & Organization
- **Naming Conventions**:
  - **Classes/Interfaces**: `PascalCase` (e.g., `PusherConnector`).
  - **Methods/Variables**: `camelCase` (e.g., `connect`, `socketId`).
  - **Files**: `kebab-case` (e.g., `pusher-connector.ts`).
  - **Constants**: `UPPER_CASE` or `camelCase` with `readonly` modifier.
- **Access Modifiers**: Use `private` or `protected` for internal implementation details.
- **Barrel Exports**: Use `index.ts` files to export public members from directories.
- **Environment Safety**: Ensure code is isomorphic where possible. Check for `window` or `document` existence before accessing browser-specific APIs.

### React Patterns
- **Custom Hooks**: Encapsulate logic in custom hooks (e.g., `useEcho`).
- **Composition**: Build complex hooks by composing smaller, focused hooks.
- **Type Safety in Hooks**: Use generics in hooks to allow users to specify payload types.
- **Effect Management**: Use `useEffect` for side effects and `useRef` for mutable values that don't trigger re-renders.
- **Dependency Arrays**: Be exhaustive and precise with dependency arrays in `useEffect` and `useCallback`.

### Directory Structure
- `src/connector/` (or similar): Implementations of core services.
- `src/contracts/` (or `src/types/`): Interfaces and type definitions.
- `src/util/`: Pure helper functions.
- `src/index.ts`: Main entry point exporting the public API.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tomnagengast)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tomnagengast)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
