## linkit

> Generates a Svelte Playground link. Ask user before use.

# Project Context: Local-First Desktop MVP

- Is a Book Mark web link maganer like a TUI

## You are building an MVP for a local-first desktop application with the following architecture:

- **Framework:** SvelteKit (using Svelte 5 Runes).
- **Runtime & Packaging:** Bun (packaged as a standalone binary).
- **Execution Model:** Operates as a local web server on `localhost`, automatically launching the default system browser upon startup.
- **Data Philosophy:** Local-first, prioritizing offline performance and local data persistence.

---

## You are able to use the Svelte MCP server, where you have access to comprehensive Svelte 5 and SvelteKit documentation. Here's how to use the available tools effectively:

## Available MCP Tools:

### 1. list-sections

Use this FIRST to discover all available documentation sections. Returns a structured list with titles, use_cases, and paths.

### 2. get-documentation

Retrieves full documentation content for specific sections.

### 3. svelte-autofixer

Analyzes Svelte code and returns issues and suggestions. MUST be used before finalizing any Svelte code.

### 4. playground-link

Generates a Svelte Playground link. Ask user before use.

## Coding Standards

Follow these principles for maintainable, simple code:

### 1. No Magic Values

- All constants in `$lib/constants.ts` with descriptive names
- Use `as const` for type safety
- Config values (retries, delays) in `APP_CONFIG`

### 2. Clean Architecture

```
stores/
  domain/     # Business logic (links, workspaces)
  infra/      # I/O (repository, logger, services)
  ui/         # UI state (filters, settings)
```

- Domain depends only on types and infra interfaces
- Infra has no dependencies
- UI depends on domain

### 3. Type Safety

- Branded types for IDs: `type LinkId = string & { readonly __brand: 'LinkId' }`
- Result type for fallible operations: `Result<T, Error>`
- Explicit return types on public functions

### 4. SOLID (Pragmatic)

- **SRP**: Functions do one thing (fetch, transform, render)
- **DIP**: Inject dependencies via options objects
- **OCP**: Extend via composition, not inheritance
- Skip Liskov/Interface Segregation unless needed

### 5. Clean Code

- Descriptive names: `fetchForWorkspace` not `load`
- Early returns over nested ifs
- Guard clauses for preconditions
- Maximum 20 lines per function (soft limit)

### 6. State Management

- Use Svelte 5 Runes: `$state`, `$derived`, `$effect`
- Encapsulate state in factory functions
- Return readonly getters, not raw state
- Optimistic updates with rollback on error

### 7. Error Handling

- Use Result type, not exceptions
- Log at boundary layers (repository, store actions)
- Rollback optimistic updates on failure

---
> Source: [ian0x-S2/linkit](https://github.com/ian0x-S2/linkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
