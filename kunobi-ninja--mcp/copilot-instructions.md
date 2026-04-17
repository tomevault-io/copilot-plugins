## mcp

> **NEVER bump the `version` field in `package.json` manually.**

# Project Guidelines

## Versioning

**NEVER bump the `version` field in `package.json` manually.**

The CI/CD release workflow uses `npm version from-git` to derive the version from git tags. The git tag (e.g., `v0.0.4`) is the single source of truth for published versions.

To release a new version:
1. Push changes to `main` and wait for CI to pass
2. Create a GitHub release with the appropriate tag (e.g., `v0.0.5`)
3. The publish workflow will automatically set the version from the tag and publish to npm

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kunobi-ninja) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
