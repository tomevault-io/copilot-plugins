## aumai-linguaforge

> Linguaforge is an open source Multi-language NLP toolkit with Indic language focus.

# Agent Instructions for Linguaforge

## Project Context
Linguaforge is an open source Multi-language NLP toolkit with Indic language focus.

## Repository Layout
- `src/aumai_linguaforge/` — package source
- `tests/` — pytest test suite
- `docs/` — user documentation
- `examples/` — runnable quickstart examples

## Capabilities
- Read and modify source files in `src/` and `tests/`
- Add new modules following existing patterns
- Update documentation and examples
- Run and fix failing tests

## Constraints
- This project is licensed under Apache 2.0
- All code must pass `ruff check src/` and `mypy src/ --strict`
- Test coverage must remain above 80%
- No external API calls without explicit user configuration
- Follow conventional commit format: `feat:`, `fix:`, `docs:`, `test:`, `refactor:`

## Development Workflow
1. Read existing tests before modifying code
2. Run `pytest tests/ -v` before committing
3. Follow conventional commit format
4. Update documentation for API changes
5. Ensure `ruff format src/` passes before any commit

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/invincible-jha)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/invincible-jha)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
