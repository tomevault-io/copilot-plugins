## bhmc-admin

> - **NEVER change the password for auth_user id=1.** This is the project owner's account. Do not reset, update, or overwrite this password under any circumstances.

# CLAUDE.md

## CRITICAL Rules

- **NEVER change the password for auth_user id=1.** This is the project owner's account. Do not reset, update, or overwrite this password under any circumstances.

- Never modify user passwords, authentication credentials, or seed data without explicit user approval. Always ask before changing any auth-related data.

## Project Overview

Golf tournament management system for Bunker Hills Men's Club (BHMC). This monorepo contains the admin API, admin web dashboard, and public-facing Next.js site (bhmc.org). It complements a separate Django backend (data.bhmc.org) that owns the database schema.

## Code Style

This project uses tab-based indentation. When editing files, always match the existing tab indentation exactly. Never convert tabs to spaces.

This project uses TypeScript with strict mode. Always handle possibly-undefined array accesses with non-null assertions or proper guards. Use the full markdown editor component (ContentEditor) for any rich text fields, not simplified alternatives.

## Data

- The Django backend at data.bhmc.org is the source of truth for data. The NestJS API reads/writes the same MySQL database using Drizzle ORM (schema defined externally — no migrations in this repo).

## Build & Development

When the UI package (`packages/ui` or similar) is modified, it must be rebuilt (check for a build/compile step that outputs to `dist/`) before changes will appear in the consuming app. Always rebuild UI packages after editing them.

## Feedback

Use the following tools / commands as feedback on your work.

`pnpm format`
`pnpm lint` (fix warnings and errors, even if pre-existing)
`pnpm test` (fix failures, even if pre-existing)
`pnpm typecheck` (fix failures, even if pre-existing)
`pnpm build` (fix failures -- ALWAYS)

When a change is made to a next app, rebuild the container.

### IMPORTANT: UX Feedback

Use the `playwright cli` skill to validate your work directly:

- Admin site: http://localhost:3100
- Public site: http://localhost:3200

## Commit Message Convention

This project uses [Conventional Commits](https://www.conventionalcommits.org/). All commit messages are validated by commitlint via a husky `commit-msg` hook.

### Format

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

### Types

| Type       | When to use                                             |
| ---------- | ------------------------------------------------------- |
| `feat`     | A new feature or capability                             |
| `fix`      | A bug fix                                               |
| `docs`     | Documentation-only changes                              |
| `style`    | Formatting, whitespace, missing semicolons, etc.        |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `perf`     | Performance improvement                                 |
| `test`     | Adding or updating tests                                |
| `ci`       | CI/CD pipeline changes                                  |
| `chore`    | Maintenance (dependencies, configs, tooling)            |
| `revert`   | Reverting a previous commit                             |

### Scopes (optional)

Use one of: `api`, `admin`, `public-next`, `domain`, `eslint-config`, `e2e`, `deps`

Omit the scope when a change spans multiple packages or is repo-wide.

### Examples

```
feat(admin): add player search to event registration
fix(api): handle null handicap in scorecard endpoint
chore(deps): update turbo to v2.9
docs: update README with deployment instructions
ci: add typecheck to GitHub Actions workflow
```

### Changelog

Generate a changelog on demand:

```sh
pnpm changelog
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/finleysg)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/finleysg)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
