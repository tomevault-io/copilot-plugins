## openskill-js

> All dependencies in `package.json` must be pinned to exact versions (no caret `^` or tilde `~` prefixes). This is important because:

# CLAUDE.md

## Dependency Versioning

All dependencies in `package.json` must be pinned to exact versions (no caret `^` or tilde `~` prefixes). This is important because:

- Prevents unintended minor/patch version changes
- Allows Renovate to manage updates deterministically
- Keeps version control clean with explicit version bumps

When updating dependencies:
1. Use `npm install package@version` to install a specific version
2. Remove any `^` or `~` prefixes in package.json
3. Verify tests pass before committing

---
> Source: [philihp/openskill.js](https://github.com/philihp/openskill.js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
