## chernycode

> This file provides instructions for Cursor's AI Agent. It follows Boris Cherny's recommendations for AI-assisted coding workflows.

# Agent Instructions - ChernyCode

This file provides instructions for Cursor's AI Agent. It follows Boris Cherny's recommendations for AI-assisted coding workflows.

## Project Context

This repository contains configurations and workflows for optimizing AI-assisted coding with Claude Code and Cursor. Use this as a reference for setting up similar workflows in other projects.

## Coding Standards

### Python Development
- Use Python 3.10+
- Format with Ruff (`ruff format .`)
- Lint with Ruff (`ruff check .`)
- Test with pytest (`pytest --cov`)
- Always include type hints
- Use Google-style docstrings
- Follow PEP 8 with max line length 120

### Naming Conventions
- Functions/variables: snake_case
- Classes: PascalCase
- Constants: UPPER_SNAKE_CASE
- Files: snake_case.py

## Git Workflow

Use Conventional Commits:
- `feat:` new feature
- `fix:` bug fix
- `docs:` documentation changes
- `refactor:` code refactoring
- `test:` adding or updating tests
- `chore:` maintenance tasks

Always:
1. Create feature branches from main
2. Run tests before committing
3. Write clear, concise commit messages

## Workflow Guidelines

### Planning
- Start complex tasks in Plan mode
- Get the plan right before implementing
- Break large tasks into smaller, focused steps

### Verification
- Always verify work with tests when available
- Run linter after making changes
- Test UI changes in browser when applicable

### Error Handling
- Use try-except with proper logging
- Provide clear error messages
- Don't silently ignore exceptions

## Key Files

| File | Purpose |
|------|---------|
| `CLAUDE.md` | Claude Code memory (project-specific) |
| `AGENTS.md` | This file - Cursor agent instructions |
| `.cursor/skills/` | Modular Cursor skills |
| `threads.md` | Boris Cherny's tips source material |

## Tools and Commands

| Task | Command |
|------|---------|
| Format code | `ruff format .` |
| Lint code | `ruff check .` |
| Fix lint issues | `ruff check --fix .` |
| Run tests | `pytest` |
| Run with coverage | `pytest --cov` |

## Preferences

- Provide concise, focused responses
- Show code examples when helpful
- Explain the "why" behind changes
- Prefer editing existing files over creating new ones
- Only create documentation when explicitly requested

---
> Source: [meleantonio/ChernyCode](https://github.com/meleantonio/ChernyCode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
