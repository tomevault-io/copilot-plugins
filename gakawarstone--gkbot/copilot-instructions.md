## gkbot

> This file contains information about using various tools and commands in this project.

# Agents

This file contains information about using various tools and commands in this project.

## Important Rules

- Do NOT use git commands directly
- Follow all coding conventions strictly

## Build/Lint/Test Commands

To run linting:

```bash
make lint
```

To run linting on a specific file:

```bash
make lint FILE=bot/core/bot.py
```

To run tests:

```bash
make test
```

To run tests on a specific file:

```bash
make test FILE=tests/test_file.py
```

To format code:

```bash
make format
```

To format a specific file:

```bash
make format FILE=bot/core/bot.py
```

## Code Style Guidelines

- Use ruff for formatting
- Follow PEP 8 naming conventions
- Use type hints for all functions
- Handle errors with specific exceptions
- Import only what's needed, at the top
- Use descriptive variable names
- Keep functions small and focused

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gakawarstone)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gakawarstone)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
