## commit-guidelines

> Guidelines for commit messages and semantic versioning


# Commit Guidelines - CRITICAL VERSION CONTROL

## ⚠️ CRITICAL WARNING - BREAKING CHANGES

**NEVER use `BREAKING CHANGE:` in commit messages unless it's a REAL breaking change that requires users to modify their code!**

The project uses semantic-release which automatically:

-   `fix:` → PATCH version (2.4.2 → 2.4.3)
-   `feat:` → MINOR version (2.4.2 → 2.5.0)
-   `BREAKING CHANGE:` → MAJOR version (2.4.2 → 3.0.0) **← AVOID THIS!**

## Commit Format

```
<type>: <emoji> <description>

[optional body]

[optional footer - NO BREAKING CHANGE unless absolutely necessary]
```

## Allowed Types

### For Version Bumps:

-   `fix: 🐛` - Bug fixes (PATCH)
-   `feat: ✨` - New features (MINOR)

### For No Version Bump:

-   `docs: 📝` - Documentation changes
-   `style: 🎨` - Code formatting, styling
-   `refactor: ♻️` - Code refactoring
-   `test: ✅` - Adding tests
-   `chore: 🔧` - Maintenance tasks
-   `ci: 🚀` - CI/CD changes

## Examples

### ✅ CORRECT:

```bash
# Minor feature (will be 2.4.2 → 2.5.0)
git commit -m "feat: ✨ add IDE selector integration"

# Bug fix (will be 2.4.2 → 2.4.3)
git commit -m "fix: 🐛 correct backup logic for single files"

# No version bump
git commit -m "docs: 📝 update README with new examples"
```

### ❌ WRONG:

```bash
# This creates MAJOR version bump (3.0.0)!
git commit -m "feat: ✨ add IDE selector

BREAKING CHANGE: Removed install command"
```

## When IS a Breaking Change?

Only use `BREAKING CHANGE:` when:

-   API changes that break existing code
-   Removing public functions/methods
-   Changing function signatures
-   Removing CLI flags that users rely on
-   Changing default behavior that breaks existing workflows

## Current Project Status

-   **Current version**: 3.0.1 (due to incorrect BREAKING CHANGE usage)
-   **Going forward**: Use only `fix:` and `feat:`
-   **Avoid**: `BREAKING CHANGE:` unless absolutely necessary

## AI Assistant Guidelines

When making commits as "Agent AI":

-   Always use `--author="Agent AI <agent.ai@assistant.local>"`
-   Prefer `fix:` over `feat:` when in doubt
-   Never use `BREAKING CHANGE:` without explicit user confirmation
-   Keep commit messages concise and descriptive

---
> Source: [tecnomanu/agent-rules-kit](https://github.com/tecnomanu/agent-rules-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
