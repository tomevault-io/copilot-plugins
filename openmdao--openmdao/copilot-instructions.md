## openmdao

> OpenMDAO provides analysis and optimization of multidisciplinary systems using the modular approach to unified derivatives.

# Project

OpenMDAO provides analysis and optimization of multidisciplinary systems using the modular approach to unified derivatives.

# Development Guidelines

- Use the "dev" environment in the pixi.toml file in the root of this project.

- Do not invoke git commands, but suggest a commit message.

# Code Style

- Use single quotes, except to define docstrings. When nesting quotes, use double quotes for the outer string.

- Documentation is in the numpy doc style.

- Do not use emojis.

# Validation

- Generated code should pass `ruff check`. Rules for ruff are established in pyproject.toml.

- Use `pixi run test <path>` to invoke tests in the relevant scope. Only run `pixi run test` if the user asks for the full test suite.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/OpenMDAO)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/OpenMDAO)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
