## orchestra

> This is the **OpenCode Boomerang** repository - a multi-agent orchestrator plugin for OpenCode that coordinates specialized AI workers to accomplish complex tasks through collaborative workflows.

# OpenCode Boomerang - Development Guidelines

## Repository Purpose

This is the **OpenCode Boomerang** repository - a multi-agent orchestrator plugin for OpenCode that coordinates specialized AI workers to accomplish complex tasks through collaborative workflows.

### Core Component: `packages/orchestrator`

The orchestrator package is the heart of the system:

- **Task API** (5 tools): `task_start`, `task_await`, `task_peek`, `task_list`, `task_cancel`
- **Worker Management**: Spawn, coordinate, and stop specialized AI workers
- **Workflow Engine**: Multi-step execution with carry/trim logic
- **Memory System**: Optional Neo4j-based memory graph for persistent context
- **Skills System**: Discovery, loading, and permission-based execution
- **Event System**: Visibility into orchestrator operations via `orchestra.*` events

## Critical Rule: NEVER EVER DELETE TESTS

**This is a hard rule. Test files are never deleted, even if they fail.**

When tests fail:
1. Examine why the test fails
2. The test expectation may be correct and the implementation wrong
3. Adjust expectations only if the implementation behavior is intentionally different
4. Never delete a test file to "make tests pass"

If a test file exists, it represents intentional test coverage. Deleting tests reduces confidence and breaks the development contract.

## Testing Standards

### No Mocks Allowed

Use **dependency injection** instead of mocking:

```typescript
// Good: Inject fake dependencies
await runWorkflow({...}, {
  resolveWorker: async (id) => id,
  sendToWorker: async (msg) => ({ success: true, response: "ok" }),
});

// Bad: Mocking external dependencies
vi.mock("some-external-module", () => ({ ... }))
```

### Test Coverage Goals

All exported functions must have unit tests coverage. Pure functions are easiest to test. Stateful components tests use injected fakes for dependencies isolation.

### Test Structure

```
test/
├── unit/           # Isolated component tests with DI
├── integration/   # Multi-component interaction tests
├── e2e/           # Full system tests (no mocks)
├── perf/          # Performance benchmarks
├── stress/        # Stress/load testing
├── optional/      # Conditional tests (Neo4j, etc)
└── helpers/       # Test utilities
```

## Code Standards

### No Comments Rule

Unless explicitly requested by the user, do not add comments to code. Let the code speak for itself.

### Minimal Changes

Prefer surgical, minimal changes over sweeping refactors. Small, focused changes are easier to review, test, and revert.

### Follow Existing Patterns

- Use the existing code style and conventions
- Follow established naming conventions
- Mimic existing patterns in adjacent code

### Clear Documentation

When making architectural decisions, document the "why" not just the "what".

## Common Commands

```bash
# Run lint
npm run lint

# Type check
npm run typecheck

# Build
npm run build

# Run tests
npm run test

# Full check (lint + typecheck + test + build)
npm run check
```

## Key Files

- `CLAUDE.md` - Core instructions for AI assistants behavior
- `packages/orchestrator/src/index.ts` - Main plugin entry point
- `packages/orchestrator/src/command/tasks.ts` - Task API implementation
- `packages/orchestrator/src/workflows/` - Workflow engine
- `packages/orchestrator/test/` - Comprehensive test suite

---
> Source: [0xSero/orchestra](https://github.com/0xSero/orchestra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
