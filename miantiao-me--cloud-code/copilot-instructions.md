## cloud-code

> A TypeScript project based on **Cloudflare Workers** and **Cloudflare Containers** (Durable Objects).

# AI Agent Development Guide (AGENTS.md)

## 1. Project Overview

A TypeScript project based on **Cloudflare Workers** and **Cloudflare Containers** (Durable Objects).
Leverages Cloudflare edge infrastructure to run and manage containerized workloads (`AgentContainer`).

**⚠️ Core Principle: All comments, logs, and commit messages MUST be in concise English.**

## 2. Build and Development Commands

### 2.1 Core Commands

| Command           | Description                        | Notes                                          |
| ----------------- | ---------------------------------- | ---------------------------------------------- |
| `pnpm dev`        | Start local development server     | Equivalent to `wrangler dev`                   |
| `pnpm start`      | Alias for `pnpm dev`               | -                                              |
| `pnpm deploy`     | Deploy to Cloudflare               | Equivalent to `wrangler deploy`                |
| `pnpm cf-typegen` | Generate Cloudflare Bindings types | Must run after modifying `wrangler.jsonc`      |

### 2.2 Code Quality Commands

| Command          | Description                 | Notes                            |
| ---------------- | --------------------------- | -------------------------------- |
| `pnpm lint`      | Run oxlint to check code    | High-performance Rust linter     |
| `pnpm lint:fix`  | Auto-fix lint issues        | Equivalent to `oxlint --fix`     |
| `pnpm fmt`       | Format code                 | Uses oxfmt (Biome fork)          |
| `pnpm fmt:check` | Check formatting compliance | Used in CI, does not modify files |

### 2.3 About Testing

> **Important**: The repository currently has **no test framework** (Jest/Vitest).
>
> - **Do not** attempt to run `pnpm test` (command does not exist).
> - Validation methods: TypeScript compiler (`pnpm cf-typegen`) + `pnpm lint` + manual code review.
> - If adding tests, Vitest is recommended (good compatibility with Wrangler).

## 3. Code Style and Conventions

### 3.1 Formatting

- **Indentation**: Tab indentation (managed by oxfmt).
- **Semicolons**: **Do not use** semicolons at end of statements (ASI style).
- **Quotes**: Prefer double quotes `"` for strings.
- **Trailing Commas**: Keep trailing commas in multi-line object/array definitions (ES2017+).
- **Code Blocks**: Always use braces `{ ... }`, even for single-line `if` statements.
- **Line Endings**: LF (`\n`), not CRLF.
- **End of File**: Keep one blank line at end of file.

### 3.2 Naming Conventions

| Type                | Convention           | Examples                              |
| ------------------- | -------------------- | ------------------------------------- |
| Files               | `camelCase`          | `container.ts`, `sse.ts`              |
| Classes             | `PascalCase`         | `AgentContainer`                      |
| Interface/Type      | `PascalCase`         | `SSEEvent`, `SSEEventHandler`         |
| Variables/Functions | `camelCase`          | `verifyBasicAuth`, `processSSEStream` |
| Constants           | `UPPER_CASE`         | `PORT`, `SINGLETON_CONTAINER_ID`      |
| Private Properties  | `_camelCase`         | `_watchPromise` (optional prefix)     |
| Boolean Variables   | `is/has/should` prefix | `isAuthorized`, `hasError`          |

### 3.3 TypeScript Best Practices

- **Strict Mode**: Strict mode is enabled (`tsconfig.json`). **Prohibit** using `any`, unless absolutely necessary with explanatory comments.
- **Type Definitions**:
  - Prefer `interface` for defining object structures.
  - Use `type` for union types and function signatures.
  - Use `satisfies` keyword to validate exported objects: `export default { ... } satisfies ExportedHandler`.
- **Environment Bindings**: Access environment variables only via `import { env } from 'cloudflare:workers'`. **Prohibit** using `process.env`.
- **Null Handling**: Prefer optional chaining `?.` and nullish coalescing `??`.
- **Type Imports**: Use `import type { ... }` to explicitly distinguish pure type imports.

### 3.4 Import Order

Maintain clear import layering (top to bottom):

```typescript
// 1. External dependencies (Cloudflare standard library, third-party packages)
import { Container } from '@cloudflare/containers'
import { env } from 'cloudflare:workers'

// 2. Internal modules (local files)
import { forwardRequestToContainer } from './container'
import { processSSEStream } from './sse'

// 3. Type imports (explicit marking)
import type { SSEEvent } from './sse'
```

## 4. Architecture and Design Patterns

### 4.1 Worker Entry and Routing

- `src/index.ts` is the Worker entry point.
- It handles HTTP request routing, basic authentication (`verifyBasicAuth`), and request forwarding.
- **Pattern**: Functions return `null` to indicate "no error/continue processing", return `Response` object to indicate "intercept/error".
  ```typescript
  // Example pattern
  function checkAuth(req): Response | null {
    if (fail) return new Response('401', ...);
    return null; // Pass
  }
  ```

### 4.2 AgentContainer (Durable Object)

- Located in `src/container.ts`.
- Inherits from `Container` (from `@cloudflare/containers`).
- **Singleton Pattern**: Uses `idFromName(SINGLETON_CONTAINER_ID)` to ensure globally unique instance for state management.
- **Lifecycle**:
  - `onStart`: Container startup hook. Used to initialize background tasks.
  - **Important**: Background tasks (like `watchContainer`) should **not be awaited** in `onStart` to avoid blocking the DO startup process, but must catch errors (fire-and-forget pattern).

### 4.3 Error Handling

- **HTTP Errors**: Prefer returning standard HTTP status code Responses (401, 403, 500).
- **External I/O**: All network requests (fetch, stream reading) must be wrapped in `try-catch`.
- **SSE Streams**: When handling Server-Sent Events, ensure proper Reader lifecycle management and gracefully exit on disconnect to avoid resource leaks.

## 5. Agent Behavior Guidelines

When modifying this codebase as an AI Agent, strictly follow these rules:

1.  **Language Consistency**:
    - All code comments must use **concise English**.
    - Git Commit Messages must use **English**.
    - **No** mixed language comments.

2.  **Non-Destructive Modifications**:
    - Without tests, be extremely cautious when modifying code.
    - After each modification, recommend running `pnpm cf-typegen` to ensure type definitions stay in sync with `wrangler.jsonc`.
    - Before modifying existing functionality, thoroughly understand its side effects.

3.  **Configuration is King**:
    - `wrangler.jsonc` is the single source of truth for infrastructure configuration.
    - Do not hardcode configuration values (like ports, secrets); use Bindings or environment variables.
    - If new variables are needed in code, must first update `wrangler.jsonc` and run `cf-typegen`.

4.  **File Operations**:
    - Prefer modifying existing files; avoid creating too many fragmented small files.
    - `worker-configuration.d.ts` is auto-generated; **do not manually modify** it.
    - Keep directory structure flat; avoid excessive nesting.

5.  **Dependency Management**:
    - Only use dependencies defined in `package.json`.
    - When introducing new libraries, first evaluate their compatibility in Cloudflare Workers Runtime (note: limited Node.js API support).
    - Prefer Web Standard APIs (Request, Response, fetch, Streams) over Node.js-specific APIs.

## 6. Toolchain

| Tool         | Description                                   |
| ------------ | --------------------------------------------- |
| **Wrangler** | Core CLI tool for deployment, dev, type generation |
| **oxlint**   | High-performance Rust linter, replaces ESLint |
| **oxfmt**    | Biome fork formatter, replaces Prettier       |
| **pnpm**     | Package manager (v10.28.1)                    |

### Verification Workflow (After Code Modifications)

```bash
# 1. Type check (if wrangler.jsonc was modified)
pnpm cf-typegen

# 2. Lint check
pnpm lint

# 3. Format check
pnpm fmt:check

# 4. Local development test
pnpm dev
```

## 7. Cursor/Copilot Integration

_(No `.cursorrules` or `.github/copilot-instructions.md` files found in this project. If added later, please supplement relevant rules here)_

---

_This document is maintained by AI Agents to unify development standards and behavior guidelines._

---
> Source: [miantiao-me/cloud-code](https://github.com/miantiao-me/cloud-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
