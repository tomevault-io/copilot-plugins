## hype

> - **NEVER commit directly to main** - Always create a feature branch and submit a PR

# Claude Code Instructions for Hype

## Critical Rules

- **NEVER commit directly to main** - Always create a feature branch and submit a PR
- **NEVER force push to main** - The branch is protected

## Git Workflow

1. Create a feature branch: `git checkout -b <type>/<description>`
2. Make changes and commit
3. Push branch: `git push -u origin <branch-name>`
4. Create PR: `gh pr create`

Branch naming conventions:
- `feat/` - New features
- `fix/` - Bug fixes
- `docs/` - Documentation changes
- `refactor/` - Code refactoring

## Project Structure

- `blog/` - Blog generator package
- `cmd/hype/cli/` - CLI commands
- `docs/` - Documentation source files
- `hype.md` - Source for README.md generation

## README Generation

The README.md is generated from `hype.md` using:

```bash
hype export -format=markdown -f hype.md > README.md
```

Always regenerate README.md after modifying `hype.md` or any included docs.

---
> Source: [gopherguides/hype](https://github.com/gopherguides/hype) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
