## fab

> Use the bus emoji 🚌 to brand the project:

# fab - Claude Code Instructions

## Branding

Use the bus emoji 🚌 to brand the project:
- README.md header
- CLI output (e.g., `🚌 fab daemon started`)
- TUI header
- Error messages where appropriate

## Project Context

fab is a coding agent supervisor - it manages multiple Claude Code instances across projects with automatic task orchestration via ticket (tk).

## Code Style

- Go 1.25
- Use standard Go project layout (cmd/, internal/)
- Prefer simplicity over abstraction
- Error messages should be actionable

## Commits

- **Atomic commits**: One logical change per commit
- **Conventional commits**: `type(scope): message`
  - Types: `feat`, `fix`, `docs`, `chore`, `refactor`, `test`
  - Scope: optional, e.g., `cli`, `tui`, `daemon`, `ticket`
- Keep subject line under 72 chars
- Reference issue IDs in body when applicable
- **Push after committing** - don't let commits pile up locally

## Releases

- Releases are CI-driven via GitHub Actions on `vX.Y.Z` tags
- Do not use `gh release` manually

## Key Conventions

- Config lives in `~/.config/fab/config.toml`
- Daemon socket at `~/.fab/fab.sock`
- Worktrees in `~/.fab/worktrees/<project>/`

---
> Source: [tessro/fab](https://github.com/tessro/fab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
