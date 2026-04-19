## clear-value

> **Never assume code works - always validate it.** Follow the iterative task execution guide for every task.

# Cursor Project Rules

## Core Principle

**Never assume code works - always validate it.** Follow the iterative task execution guide for every task.

## Required Reading

Before starting any task, agents MUST read:

- `docs/iterative-task-execution.md` - Complete workflow for task execution
- `docs/technical-architecture.md` - System architecture
- `docs/development-setup.md` - Development environment setup
- `docs/api-specification.md` - API contracts (for API work)
- `docs/data-models.md` - Data structures (for data work)
- `docs/design-system.md` - Design guidelines (for frontend work)

## Task Execution Workflow

Follow the 8-phase workflow from `docs/iterative-task-execution.md`:

1. **Understanding & Planning** - Read requirements, understand current state, plan implementation
2. **Environment Setup** - Verify prerequisites, check environment
3. **Iterative Implementation** - Make small changes, validate after each
4. **Testing & Validation** - Manual testing, integration testing, edge cases
5. **Build Verification** - Full build, production build test
6. **Code Quality Checks** - Linting, type checking, code review
7. **Completion Verification** - Acceptance criteria, functionality verification
8. **Final Validation** - Clean build, integration verification

## Validation Requirements

After EVERY code change, you MUST:

1. Run type checking: `pnpm typecheck`
2. Run linting: `pnpm lint`
3. Build the affected package: `pnpm --filter <package> build`
4. Test the change manually (run dev server, test in browser/API)
5. Fix any errors before proceeding

## Project Structure

- `apps/api/` - Fastify backend (TypeScript)
- `apps/web/` - React frontend (TypeScript, Vite)
- `packages/shared/` - Shared types and schemas (TypeScript)
- `docs/` - Project documentation

## Code Style

- **TypeScript**: Strict mode, use shared types from `@clearvalue/shared`
- **Naming**: camelCase for variables, PascalCase for types/classes
- **Error Handling**: Always return structured error responses
- **Validation**: Use Zod schemas for all inputs
- **Comments**: Document complex algorithms and business logic
- **No `any` types**: Use proper types, avoid `any` unless absolutely necessary

## Shared Package Workflow

When modifying `packages/shared/`:

1. Make changes to types/schemas
2. Rebuild: `pnpm --filter @clearvalue/shared build`
3. Rebuild dependent packages: `pnpm --filter @clearvalue/api build && pnpm --filter @clearvalue/web build`
4. Verify no breaking changes

## Testing Requirements

- Test all success cases
- Test all error cases
- Test edge cases (null, empty, boundary values)
- Test integration with existing code
- Verify no regressions

## Build Requirements

Before marking task complete:

- [ ] All packages build successfully: `pnpm build`
- [ ] Type checking passes: `pnpm typecheck`
- [ ] Linting passes: `pnpm lint`
- [ ] Manual testing complete
- [ ] All acceptance criteria met
- [ ] No regressions introduced

## Error Handling

- Always handle errors gracefully
- Return appropriate HTTP status codes (for API)
- Include error codes and helpful messages
- Log errors for debugging
- Never throw unhandled errors

## Dependencies

- Check if dependency already exists before adding
- Use `pnpm add <package>` in correct workspace
- Document new dependencies if significant

## Git Workflow

- Use conventional commits: `feat:`, `fix:`, `refactor:`, `docs:`, etc.
- Commit frequently with clear messages
- Don't commit broken code

## Completion Checklist

A task is NOT complete until:

- [ ] All code compiles without errors
- [ ] All linting passes
- [ ] All type checking passes
- [ ] Code has been manually tested
- [ ] All acceptance criteria are met
- [ ] No regressions introduced
- [ ] Documentation updated (if needed)
- [ ] Code follows project patterns
- [ ] Error handling is comprehensive
- [ ] Build succeeds in production mode

## Important Reminders

- **DO NOT** skip validation steps
- **DO NOT** assume code works without testing
- **DO NOT** accumulate errors - fix immediately
- **DO** validate after every change
- **DO** test manually
- **DO** follow the iterative workflow
- **DO** reference existing code patterns

## Quick Reference Commands

```bash
# After each change
pnpm typecheck && pnpm lint && pnpm build

# Full validation
pnpm install
pnpm --filter @clearvalue/shared build
pnpm typecheck
pnpm lint
pnpm build
pnpm dev  # Test manually

# Test API
curl http://localhost:3000/api/v1/health

# Test Frontend
# Open http://localhost:5173 in browser
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tmorse01) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
