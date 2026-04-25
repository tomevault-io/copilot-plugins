## typed-handler

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`typed-handler` is a TypeScript library that provides a fluent, type-safe API for building request handlers with automatic validation and framework-agnostic design. The library allows developers to create handlers that work with any validation library (Zod, Joi, Yup, etc.) and any web framework (Express, Fastify, Hono, etc.).

**Key Design Principles:**
- **Type Safety First**: All types are automatically inferred without manual annotations
- **Zero Vendor Lock-in**: Support any validator, any framework
- **Immutability**: Each builder method returns a new handler instance
- **Zero Dependencies**: Core library has no runtime dependencies

## Development Commands

### Package Manager
This project uses **pnpm** (version 8.15.0). Always use pnpm, never npm or yarn.

```bash
# Install dependencies
pnpm install

# Set up git hooks (run after install)
pnpm prepare
```

### Development
```bash
# Run development server (watches examples/express-zod/index.ts)
pnpm dev

# Type checking
pnpm type-check
```

### Testing
```bash
# Run tests (Vitest)
pnpm test

# Run tests in watch mode
pnpm test:watch

# Run tests with coverage (requires 90% coverage threshold)
pnpm test:coverage

# Run type tests (tsd)
pnpm test:types
```

### Building
```bash
# Build library (uses tsup)
pnpm build

# Output: dist/ directory with CommonJS, ESM, and TypeScript declarations
# Entry points: src/index.ts and src/adapters/index.ts
```

### Code Quality
```bash
# Lint (Biome)
pnpm lint

# Lint and auto-fix
pnpm lint:fix

# Format code
pnpm format
```

### Git Hooks
- **Pre-commit**: Runs `pnpm lint-staged` (lints staged files with Biome)
- **Pre-push**: Runs `pnpm type-check && pnpm test --run`

### Publishing
```bash
# Create changeset
pnpm changeset

# Version packages
pnpm version

# Release (builds and publishes)
pnpm release
```

## Architecture

### Core Components

1. **Handler Class** (`src/handler.ts`): Main builder class using immutable chain construction. Each method clones the handler and returns a new instance.

2. **Validator System** (`src/validators/`): Auto-detects validation libraries (Zod, Joi, Yup) and provides adapter registration for custom validators.

3. **Framework Adapters** (`src/adapters/`): Bridge handlers to web frameworks (Express, Fastify, Hono, Raw).

4. **Type System** (`src/types.ts`): TypeScript types for full type inference throughout the handler chain.

5. **Configuration** (`src/config.ts`): Global and per-handler configuration with pluggable logger interface.

### Execution Flow

```
Input Data → Validate Input → Run Middleware (build context) →
Execute Handler → Validate Output → Return Result
```

Each step:
- Input validation is configured via `validateInput` (default: true)
- Middleware accumulates context with type safety
- Handler receives typed input and context
- Output validation is configured via `validateOutput` (default: off in production)

### Multi-Input Detection

The library detects if input schema is an object with `body`, `query`, `params`, or `headers` keys. When detected, framework adapters extract the appropriate data from the request object.

### Immutability Pattern

Critical implementation detail: All builder methods (`.input()`, `.use()`, `.handle()`, `.output()`) must:
1. Clone the current handler via `this.clone()`
2. Modify the clone
3. Return the clone with updated types

This ensures handlers can be reused and composed without mutations.

## Code Structure

```
src/
├── index.ts                 # Main exports
├── handler.ts               # Handler class (core builder)
├── types.ts                 # Core types and type inference
├── config.ts                # Configuration system
├── validators/
│   └── registry.ts          # Custom validator registration
├── adapters/
│   ├── index.ts             # Adapter exports
│   ├── express.ts           # Express integration
│   ├── fastify.ts           # Fastify integration
│   ├── hono.ts              # Hono integration
│   └── raw.ts               # Raw handler (custom integration)
├── errors/
│   ├── index.ts
│   ├── validation.ts        # ValidationError
│   └── handler.ts           # HandlerError
└── utils/
    └── response.ts          # ResponseObject helpers
```

## Implementation Status

**Current Status**: Core handler implementation is complete, including all builder methods, validation system, and framework adapters.

The technical specification (`TECHSPEC.md`) provides the complete design. Reference it when implementing features.

## Testing Strategy

- **Unit tests**: Core handler functionality, validators, execution flow
- **Integration tests**: Framework adapters with real HTTP requests
- **Type tests**: Type inference scenarios using `tsd`
- **Coverage target**: >90% for lines, functions, branches, statements

Test files go in `tests/` directory:
```
tests/
├── unit/
├── integration/
└── types/
```

## Type Safety Requirements

When implementing or modifying code:

1. **Preserve type inference**: Handler methods must return correctly typed handlers
2. **Context accumulation**: `.use()` must merge context types: `Handler<TInput, TContext & TNewContext, TOutput>`
3. **Input inference**: Support both schema-based and explicit type parameter modes
4. **Type tests**: Add `tsd` tests for any new type inference scenarios

## Common Patterns

### Creating a Handler
```typescript
const h = handler()
  .input(zodSchema)           // Infer input type from schema
  .use(async () => ({ user })) // Add context
  .handle(async (input, ctx) => result) // Handler function
  .output(zodSchema);          // Validate output
```

### Framework Adapter Pattern
```typescript
// Adapters check for multi-input and extract appropriately
const input = handler.expectsMultiInput()
  ? { body: req.body, query: req.query, params: req.params, headers: req.headers }
  : req.body;

const result = await handler.execute(input, { req, res });
```

### Configuration
- Global: `configure({ validateOutput: false })`
- Per-handler: `handler({ logger: customLogger })`
- Default logger is no-op (safe to call without checks)

## Biome Linting

This project uses Biome for linting and formatting. Common lint ignores in the codebase:

- `noExplicitAny`: Required for registry and type inference flexibility
- `noStaticOnlyClass`: For factory pattern placeholders
- `complexity/noStaticOnlyClass`: For handler factory

## Important Notes

- **Never use npm or yarn**, always use pnpm
- **Preserve immutability** in handler methods (clone before modify)
- **Reference tech spec** (`TECHSPEC.md`) for detailed design decisions
- **Test coverage is enforced** at 90% - ensure new code is well tested
- **Type safety is critical** - all handler chains must have full type inference

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rledford) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
