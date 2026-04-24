## config-eslint

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**Use Context7 MCP server for up-to-date library documentation.**

## Project Overview

`@smooai/config-eslint` is the shared ESLint configuration package used across all SmooAI projects. It exports framework-specific configs for consistent linting across the organization.

### Exports

- `index.js` -- Base ESLint config
- `next.js` -- Next.js projects
- `react.js` -- React projects
- `remix.js` -- Remix projects (legacy)
- `vite.js` -- Vite projects

---

## Git Workflow -- Worktrees

All feature work MUST happen in git worktrees. The main worktree at `~/dev/smooai/config-eslint/` must ALWAYS stay on the `main` branch.

### Creating a worktree

```bash
cd ~/dev/smooai/config-eslint
git worktree add ../config-eslint-SMOODEV-XX-short-desc -b SMOODEV-XX-short-desc main
cd ../config-eslint-SMOODEV-XX-short-desc
pnpm install
```

### Branch naming

```
SMOODEV-XX-short-description
```

### Commit messages

Always prefix with the Jira ticket. Explain **why**, not just what:

```
SMOODEV-XX: Add survey validation to prevent duplicate submissions
```

### Merging to main

```bash
cd ~/dev/smooai/config-eslint
git checkout main && git pull --rebase
git merge SMOODEV-XX-short-desc --no-ff
git push
```

### Cleanup after merge

```bash
git worktree remove ~/dev/smooai/config-eslint-SMOODEV-XX-short-desc
git branch -d SMOODEV-XX-short-desc
```

---

## Commands

```bash
pnpm install          # Install dependencies
pnpm lint             # Lint with oxlint
pnpm format           # Format with oxfmt
pnpm format:check     # Check formatting without fixing
```

There is no build or test step for this package -- it exports plain JS config files.

---

## Changesets & Versioning

Always add changesets when the package changes:

```bash
pnpm changeset        # Interactive changeset creation
```

Release is automated via GitHub Actions.

---

## CI / GitHub Actions

PR checks run on every PR to `main`. CI must be green before merging.

```bash
gh run list                          # List recent workflow runs
gh run view <run-id> --log-failed    # View failed step logs
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SmooAI) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
