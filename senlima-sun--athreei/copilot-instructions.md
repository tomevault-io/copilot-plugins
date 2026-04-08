## athreei

> Quick reference for AI agents performing quality assurance tasks in this monorepo.

# AGENTS.md

Quick reference for AI agents performing quality assurance tasks in this monorepo.

## Quick Commands

```bash
# Quality gates (run ALL before committing)
bun run typecheck:all    # Type checking
bun run test             # All tests (Vitest)
bun run lint             # ESLint
bun run format:check     # Prettier

# Fix issues
bun run lint:fix         # Auto-fix lint
bun run format           # Auto-format

# Single file/package testing
npx vitest run <path>    # Run specific test
bun run test:watch       # Watch mode
bun run test:sqlite      # SQLite tests only
```

**⚠️ NEVER use `bun test`** — incompatible with Vitest APIs (`vi.hoisted`, `vi.mocked`, jsdom).

## Quality Gate Checklist

Before any commit:

| Gate   | Command                 | Requirement |
| ------ | ----------------------- | ----------- |
| Types  | `bun run typecheck:all` | Zero errors |
| Tests  | `bun run test`          | All pass    |
| Lint   | `bun run lint`          | No errors   |
| Format | `bun run format:check`  | Clean       |

All four must pass. No exceptions.

## Testing

### Framework: Vitest

- Node environment default, `jsdom` for browser simulation
- Globals enabled (`describe`, `it`, `expect` without imports)

### Coverage Targets

| Category       | Minimum |
| -------------- | ------- |
| API routes     | 80%     |
| Business logic | 90%     |
| Utilities      | 70%     |
| UI components  | 60%     |

### Must Test

- Happy path for all public APIs
- Error responses: 400, 401, 403, 404, 500
- Authentication/authorization checks
- Zod validation (valid and invalid inputs)
- Edge cases: null, empty, boundary values

### Skip Testing

- Private functions / implementation details
- Third-party library internals
- Trivial getters/setters

### Hono Route Testing Pattern

```typescript
const { default: routes } = await import("../../routes/api-keys")
const app = new Hono()
app.onError(testErrorHandler)
app.route("/api/api-keys", routes)

const response = await app.request("/api/api-keys/ep_123/keys")
expect(response.status).toBe(200)
```

### Mocking Pattern

```typescript
vi.mock("../../lib/db", () => ({
  getDb: vi.fn(() => mockDb),
}))

beforeEach(() => vi.clearAllMocks())
```

## Type Checking

### Rules

- No `any` without justification comment
- Prefer `unknown` + type guards over `any`
- No `@ts-ignore` — use `@ts-expect-error` with linked issue if unavoidable
- No `!` non-null assertions — handle nullability properly

### Common Fixes

```typescript
// Bad: any
const data: any = await fetch(url)

// Good: unknown + guard
const data: unknown = await fetch(url)
if (isUser(data)) {
  /* typed */
}

// Bad: assertion
const user = data as User

// Good: validation
const user = UserSchema.parse(data)
```

## Code Style

### Prettier Config

- No semicolons
- Double quotes
- 2-space indentation
- Trailing commas (ES5)

### Naming

| Type       | Convention  | Example           |
| ---------- | ----------- | ----------------- |
| Variables  | camelCase   | `userId`          |
| Constants  | UPPER_SNAKE | `MAX_RETRIES`     |
| Functions  | camelCase   | `getUserById`     |
| Types      | PascalCase  | `UserWithRole`    |
| Files      | kebab-case  | `user-service.ts` |
| Components | PascalCase  | `UserProfile.tsx` |

### Unused Variables

Prefix with underscore:

```typescript
function handler(_req: Request, res: Response) {
  // _req intentionally unused
}
```

## Package-Specific Commands

```bash
# Database
cd packages/db && bun run migrate

# Gateway binary
cd packages/gateway && bun run build:binary

# Desktop app
cd apps/desktop && bun run tauri:dev

# API server
cd apps/api && bun run dev

# Platform
cd apps/platform && bun run dev
```

## Workflow

1. **Plan** — Break task into steps
2. **Implement** — Code incrementally
3. **Test** — Write/run tests
4. **Review** — Use code-reviewer agent
5. **Verify** — Run all quality gates
6. **Commit** — Atomic commits, conventional format

## Commit Format

```
<type>(<scope>): <subject>

Types: feat, fix, refactor, test, docs, chore
Scope: package or app name
```

Examples:

- `feat(gateway): add trace collection`
- `fix(api): handle null user ID`
- `test(db): add migration tests`

## When Stuck

### Type Errors

1. Read the full error message
2. Check if type definitions exist (`@types/*`)
3. Verify import paths
4. Check `tsconfig.json` includes

### Test Failures

1. Run single test: `npx vitest run <file>`
2. Check mock setup (cleared in `beforeEach`?)
3. Verify test isolation (no shared state)
4. Check async/await handling

### Lint Errors

1. Run `bun run lint:fix` first
2. Check `.eslintrc` for rule config
3. Unused imports → remove them
4. Console calls → use structured logger (see Logging section)

## Logging

Use the structured logger from `@athreei/shared` instead of `console.log/error/warn`:

```typescript
import { createLogger, honoLogger } from "@athreei/shared"

// Create service logger
const logger = createLogger({
  service: "api",
  level: process.env.LOG_LEVEL || "info",
  pretty: process.env.NODE_ENV !== "production",
})

// Use structured logging
logger.info("Operation completed", { userId, duration })
logger.warn("Rate limit approaching", { current: 95, limit: 100 })
logger.error("Database error", { error, query })

// For Hono apps, use middleware
app.use("*", honoLogger({ logger }))

// Access request-scoped logger in routes
const log = c.get("logger")
log.info("Processing request") // Includes requestId
```

### When to use each level

| Level   | Use for                                     |
| ------- | ------------------------------------------- |
| `debug` | Detailed debugging (disabled in production) |
| `info`  | Normal operations, startup messages         |
| `warn`  | Recoverable issues, deprecation notices     |
| `error` | Errors requiring attention                  |

## Security Checklist

- [ ] Input validated with Zod at boundaries
- [ ] Authorization checked before data access
- [ ] No secrets in code or logs
- [ ] No SQL injection (use Drizzle ORM)
- [ ] No XSS (sanitize user content)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/senlima-sun)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/senlima-sun)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
