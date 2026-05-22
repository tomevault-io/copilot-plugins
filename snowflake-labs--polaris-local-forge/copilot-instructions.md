## workflow

> Workflow conventions - commits, branching, planning


# Workflow Conventions

## Conventional Commits

Use semantic commit messages:

- `feat:` new feature
- `fix:` bug fix
- `docs:` documentation
- `refactor:` code refactoring
- `test:` tests
- `chore:` maintenance

## Issue Fixing Workflow

1. Create branch: `issues/<issue-context>` (e.g., `issues/k3d-cluster-timeout`)
2. Use **Workspace Tree** in Cursor to understand structure before fixing
3. Make focused, atomic commits
4. Reference issue in commit message when applicable

## Planning First

**CRITICAL:** When you see questions, "WDYT", or ambiguous requests:

1. **Switch to Plan mode** - Don't run agent mode directly
2. **Understand scope** - Read relevant code, check dependencies
3. **Propose approach** - Outline steps before executing
4. **Get approval** - Wait for user confirmation

Plan → Execute pattern ensures better outcomes than diving straight into changes.

---
> Source: [Snowflake-Labs/polaris-local-forge](https://github.com/Snowflake-Labs/polaris-local-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
