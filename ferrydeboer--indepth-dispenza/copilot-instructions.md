## indepth-dispenza

> This file provides guidance to Claude Code (claude.ai/code) when working with this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with this repository.

## Development Workflow

This project follows **Trunk Based Development** - commit directly to `main`.

## Commit Convention

Use [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(#<issue>): <description>
```

**Types:**
- `feat` - new feature
- `fix` - bug fix
- `refactor` - code change that neither fixes a bug nor adds a feature
- `test` - adding or updating tests
- `docs` - documentation only
- `chore` - maintenance tasks

**Scope:** When work relates to a GitHub issue, include the issue number as scope: `(#123)`

**Examples:**
```
feat(#2): add versionLabel field to CosmosStoredLlmDocument
fix(#1): populate taxonomyVersion during analysis
refactor: extract video ID parsing to helper
test(#2): add unit tests for playlist filtering
```

## Bug Fixing Workflow

When fixing bugs linked to GitHub issues, follow the two-commit rule:

1. **First commit: Reproduce the bug**
   - Write a failing test that demonstrates the bug
   - Commit message: `test(#issue): reproduce - {brief description}`

2. **Second commit: Fix the bug**
   - Implement the fix so the test passes
   - Commit message: `fix(#issue): {brief description}`

### Example

```bash
# Write failing test
git commit -m "test(#1): reproduce - taxonomyVersion is always null"
# Implement fix
git commit -m "fix(#1): populate taxonomyVersion during analysis"
# Push to main
git push
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ferrydeboer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
