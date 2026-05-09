## ci-enforcement

> Enforce running CI checks after code changes


# CI Enforcement Rule

**Always run `mise run ci` after making code changes and ensure it passes before completing any task.**

## Required Workflow

1. **After any code changes**, run:
   ```bash
   mise run ci
   ```

2. **If CI fails**, fix all issues before proceeding:
   - **Linting errors**: Fix code quality issues reported by ESLint
   - **Formatting issues**: Let Prettier auto-fix formatting
   - **Test failures**: Ensure all tests pass
   - **Type errors**: Resolve TypeScript compilation issues

3. **Only complete the task when CI passes completely**

## Common Issues to Fix

### ESLint Linting Errors
- **Unused variables**: Remove or prefix with underscore
  ```typescript
  // ❌ DON'T: Unused variable
  const unusedVar = 42;

  // ✅ DO: Remove or prefix with underscore
  const _unusedVar = 42; // Explicitly marked as intentionally unused
  ```

- **Formatting**: Let Prettier auto-fix with `pnpm format:fix`
- **Linting**: Let ESLint auto-fix with `pnpm lint:fix`

### Test Issues
- Ensure all mocks are properly configured
- Verify test expectations match actual behavior
- Check that setup/teardown is correct

## Commands Reference

- `mise run ci` - Run full CI pipeline
- `pnpm check:fix` - Run Prettier and ESLint checks and auto-fix
- `pnpm format:fix` - Run Prettier formatting auto-fix
- `pnpm lint:fix` - Run ESLint checks and auto-fix
- `pnpm test` - Run tests only
- `pnpm build` - Build the project

## Non-Negotiable Rule

**Never mark a task as complete if `mise run ci` fails.** Always fix the issues first.

---
> Source: [Real1tyy/Nexus-Properties](https://github.com/Real1tyy/Nexus-Properties) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
