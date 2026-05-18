## commitlint

> Commitlint rules for conventional commit messages


# Commitlint Rules

All commit messages must follow the [Conventional Commits](https://www.conventionalcommits.org/) specification.

## Commit Message Format

```
<type>(<scope>): <subject>

[optional body]

[optional footer(s)]
```

### Structure Rules

| Part | Rule | Example |
|------|------|---------|
| **type** | Required, lowercase | `feat`, `fix`, `docs` |
| **scope** | Optional, lowercase, in parentheses | `(auth)`, `(calendar)` |
| **subject** | Required, lowercase start, no period at end | `add user authentication` |
| **body** | Optional, blank line after subject | Detailed explanation |
| **footer** | Optional, for breaking changes/issues | `BREAKING CHANGE:`, `Fixes #123` |

## Allowed Types

| Type | Description | Bumps |
|------|-------------|-------|
| `feat` | New feature | Minor |
| `fix` | Bug fix | Patch |
| `docs` | Documentation only | - |
| `style` | Code style (formatting, semicolons) | - |
| `refactor` | Code change that neither fixes nor adds | - |
| `perf` | Performance improvement | Patch |
| `test` | Adding/updating tests | - |
| `build` | Build system or dependencies | - |
| `ci` | CI/CD configuration | - |
| `chore` | Other changes (tooling, config) | - |
| `revert` | Reverts a previous commit | - |

## Allowed Scopes (Project-Specific)

```
auth, drive, docs, sheets, calendar, gmail,
contacts, youtube, slides, tasks, server, types, deps
```

## Rules

### Header Rules

- **type-enum**: Type must be one of the allowed types
- **type-case**: Type must be lowercase
- **type-empty**: Type cannot be empty
- **scope-case**: Scope must be lowercase
- **subject-case**: Subject must start with lowercase
- **subject-empty**: Subject cannot be empty
- **subject-full-stop**: Subject cannot end with period
- **header-max-length**: Header max 100 characters

### Body Rules

- **body-leading-blank**: Body must have blank line before it
- **body-max-line-length**: Body lines max 100 characters

### Footer Rules

- **footer-leading-blank**: Footer must have blank line before it
- **footer-max-line-length**: Footer lines max 100 characters

## Examples

### Simple commits

```bash
# Feature
git commit -m "feat(calendar): add recurring event support"

# Bug fix
git commit -m "fix(auth): handle token refresh failure"

# Documentation
git commit -m "docs: update README with setup instructions"

# Chore
git commit -m "chore(deps): update googleapis to v145"
```

### With body

```bash
git commit -m "feat(gmail): add email threading support

Implements conversation view for email threads.
Groups related messages by thread ID and displays
them in chronological order."
```

### Breaking change

```bash
git commit -m "feat(auth)!: change token storage location

BREAKING CHANGE: Tokens now stored in XDG directories.
Users must re-authenticate after updating."
```

### Multiple scopes

```bash
# Use comma-separated scopes sparingly
git commit -m "refactor(drive,docs): extract shared file utilities"
```

### Referencing issues

```bash
git commit -m "fix(sheets): handle empty cell values

Fixes #42"
```

## Invalid Examples

```bash
# ❌ Type not lowercase
git commit -m "Feat(auth): add login"

# ❌ Missing type
git commit -m "add new feature"

# ❌ Subject starts with capital
git commit -m "feat: Add new feature"

# ❌ Subject ends with period
git commit -m "feat: add new feature."

# ❌ Header too long (over 100 chars)
git commit -m "feat(calendar): add support for creating recurring events with complex recurrence rules and multiple attendees"

# ❌ Invalid type
git commit -m "feature(auth): add login"
```

## Tips

1. **Keep subjects concise** - Use imperative mood ("add" not "added")
2. **Use body for context** - Explain *why*, not just *what*
3. **Reference issues** - Link to related issues/PRs in footer
4. **One logical change per commit** - Atomic commits are easier to review
5. **Breaking changes** - Use `!` after type/scope and add `BREAKING CHANGE:` footer

---
> Source: [quinnjr/google-mcp](https://github.com/quinnjr/google-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
