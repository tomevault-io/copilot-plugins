## middle-earth

> Conventional Commit message standards with Middle Earth scopes


# Commit messages (Middle Earth)

## Format

- **Structure**: `<type>(optional scope): <description>`

## Requirements

- **MUST** follow Conventional Commits
- **MUST** use imperative mood (add, fix, update; not added, fixed, updated)
- **MUST** start description with lowercase letter
- **MUST NOT** end description with a period
- **MUST** keep full header <= 100 characters
- **SHOULD** keep description <= 72 characters

## Allowed types

`feat`, `fix`, `refactor`, `docs`, `test`, `style`, `perf`, `build`, `ci`, `chore`, `revert`

## Middle Earth scopes

- **`config`**: Root-level configuration files (nx.json, tsconfig.base.json, ESLint, commitlint)
- **`lib`**: Libraries in `libs/publishable/`
- **`release`**: Versioning, releases, publishing
- **`ai`**: AI instructions and rules

## Examples

```
feat(lib): add Button component to UI library
chore(config): update ESLint rules for TypeScript files
chore(release): bump eslint-config-carrot to 0.9.0
refactor(lib): simplify logger initialization
docs(ai): update testing rule for Jest patterns
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/carrot-foundation) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
