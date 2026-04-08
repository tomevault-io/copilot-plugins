## chronicler

> Welcome to Chronicler. These notes are for coding agents working in this repository.

# Agent Guidelines

Welcome to Chronicler. These notes are for coding agents working in this repository.

## Spec-Driven Development (required)

Chronicler uses **Spec-Driven Development (SDD)**. The `/specs` directory is the source of truth for
how the system behaves **today**.

**Rules:**
- Only **product features** must start with a new or updated spec in `/specs`.
- Bug fixes for existing specs **must add clarifications** or corrections to the relevant spec.
- Specs should describe current behavior in clear, testable terms (not future intent).
- All specs must follow the structure in `specs/template.md` (copy it and fill in the sections).
- If code and spec diverge, update the spec to match reality **before** shipping code changes.

## Development conventions

- Keep changes small and focused.
- Prefer the Makefile targets when available: `make test`, `make lint`, `make migrate`.
- Format Go code with `gofmt` and keep imports tidy.
- Document new configuration keys and API changes in the appropriate spec.
- Use semantic commit messages (e.g. `feat: add export signing`, `fix: handle empty batch`, `docs: update spec template`).
- Review `CONTRIBUTING.md` for contribution workflow and `CONTEXT.md` for repository context before making changes.

## Review checklist

- Spec updated or added for the change.
- Tests updated/added when behavior changes.
- No secrets or environment-specific values committed.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tank-studio-tech)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tank-studio-tech)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
