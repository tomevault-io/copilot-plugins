## obsidian-kindle-plugin

> Working conventions and decisions for AI-assisted development on this repo.

# AGENTS.md — obsidian-kindle-plugin

Working conventions and decisions for AI-assisted development on this repo.

---

## Project Overview

- **What**: Obsidian plugin for syncing Kindle highlights/notes into an Obsidian vault
- **Tech**: TypeScript, Rollup bundler, Obsidian Plugin API
- **Stars**: 1,200+ | **Forks**: 86 | **Status**: Revived (Feb 2026 after ~3 year hiatus)
- **Owner**: hadynz (GitHub SSH) — note: `gh` CLI may be authenticated as a different account

## Quick Reference

| Command         | Expected               |
| --------------- | ---------------------- |
| `npm run lint`  | 0 errors (warnings OK) |
| `npm run test`  | All pass (127+ tests)  |
| `npm run build` | `dist/main.js` (~1MB)  |
| Node version    | 20.x only              |

## Documentation

| Document                                           | Contents                                                                                                      |
| -------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| [Architecture](docs/architecture.md)               | Key files, sync behavior, properties format, Amazon regions                                                   |
| [Build & Release](docs/build-and-release.md)       | Build commands, versioning (3-file bump), release process, local testing, CI/CD, git conventions, GitHub auth |
| [Decisions](docs/decisions.md)                     | Deferred work, excluded PRs, open design questions                                                            |
| [Engineering Journal](docs/engineering-journal.md) | Session-by-session learnings, gotchas, and metrics                                                            |

## Critical Conventions

- **Version bumps** touch 3 files: `package.json`, `manifest.json`, `versions.json`
- **Merges** use `--no-ff` (preserves PR history)
- **Batch work** (3+ PRs) uses an integration branch, not direct-to-master
- **Pre-commit hooks** run lint-staged. Never bypass with `--no-verify`
- **`gh` CLI auth** may differ from SSH auth. Verify with `gh auth status` at session start
- **Manual validation** required for all new user-facing features. Owner tests locally in Obsidian vault before release.

---
> Source: [hadynz/obsidian-kindle-plugin](https://github.com/hadynz/obsidian-kindle-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
