## google-docstring-parser

> This document contains rules and guidelines for Claude AI when working on this project.

# Claude AI Rules

This document contains rules and guidelines for Claude AI when working on this project.

## Communication

- Do not create a summary document on every fix

## Philosophy

Our development approach follows this priority order:

1. **Delete** - First, delete as much code as possible
2. **Simplify** - Second, simplify what remains
3. **Optimize** - Third, optimize if needed

## Python Development

- We use Python 3.10 typing
- For tests we use pytest and parametrize
- We use Google type docstrings
- To check syntax we do not run ruff, flake8, but just run `pre-commit run --all-files`

## Testing

- We use extensive testing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ternaus)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ternaus)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
