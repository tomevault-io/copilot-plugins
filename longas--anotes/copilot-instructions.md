## anotes

> Apple Notes CLI for AI agents.

# anotes

Apple Notes CLI for AI agents.

## Rules

- Always use Context7 MCP when I need library/API documentation, code generation, setup or configuration steps without me having to explicitly ask.
- Always run `npm run typecheck` and `npm run fix` after finishing code changes.

## Releasing

This project uses [semantic-release](https://github.com/semantic-release/semantic-release) for automated versioning and npm publishing.

### How it works

1. semantic-release analyzes commit messages on `main` to determine the next version bump
2. A GitHub Actions workflow (`.github/workflows/release.yml`) runs on every push to `main`
3. If there are releasable commits, it automatically bumps the version, creates a GitHub Release with release notes, and publishes to npm

### Commit message format

Commits must follow [Conventional Commits](https://www.conventionalcommits.org/):

- `fix: description` — triggers a **patch** release (0.0.x)
- `feat: description` — triggers a **minor** release (0.x.0)
- `feat!: description` or a commit with `BREAKING CHANGE:` in the footer — triggers a **major** release (x.0.0)
- `chore:`, `docs:`, `style:`, `refactor:`, `test:`, `ci:` — **no release**

Examples:
```
fix: handle empty note body in search results
feat: add folder listing command
feat!: rename --format flag to --output
```

---
> Source: [longas/anotes](https://github.com/longas/anotes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
