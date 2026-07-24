## zcmds

> Before considering any issue or task completely done, you MUST:

# Cursor AI Rules

## Completion Requirements

Before considering any issue or task completely done, you MUST:

1. **Run linting**: Execute `bash lint` to ensure code quality and style compliance
2. **Run tests**: Execute `bash test` to verify all tests pass

These commands are mandatory and must be executed successfully before marking any work as complete.

## Workflow

1. Make your code changes
2. Run `bash lint` and fix any linting issues
3. Run `bash test` and ensure all tests pass
4. Only then consider the task complete

## Notes

- Both commands must exit with status code 0 (success)
- If either command fails, the issue is NOT complete
- Fix any issues found by these commands before proceeding
- These commands are defined in the project's testing infrastructure

---
> Source: [zackees/zcmds](https://github.com/zackees/zcmds) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
