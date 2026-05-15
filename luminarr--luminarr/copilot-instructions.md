## luminarr

> **Before creating any GitHub release**, you MUST query GitHub for the current latest release:

# Luminarr — Claude Code Rules

## Releases

**Before creating any GitHub release**, you MUST query GitHub for the current latest release:

```sh
gh release list --repo luminarr/luminarr --limit 5
```

Use the actual latest version from GitHub to determine the next version number. **Never rely on local `git tag`** — local tags can be out of sync with GitHub.

## GitHub

All `gh` commands MUST target `luminarr/luminarr`:

```sh
gh <command> --repo luminarr/luminarr
```

## Branching

**Always work on a feature branch** — never commit directly to `main`. Create a branch before starting new work:

```sh
git checkout -b feat/my-feature
```

Merge to `main` via PR or fast-forward after work is complete and tests pass.

## Code Quality

- Run `make check` before every push (golangci-lint + tsc --noEmit).
- One logical unit per commit.
- Frontend tests: `cd web/ui && npm test` must pass before pushing frontend changes.

---
> Source: [luminarr/luminarr](https://github.com/luminarr/luminarr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->
