## tiged

> All commits MUST follow **Conventional Commits**.

# Agent guidelines

## Commits (required)

All commits MUST follow **Conventional Commits**.

Format:

- `type(scope): summary`
- Optional body for context
- Use `!` or a `BREAKING CHANGE:` footer for breaking changes

Allowed `type` (typical): `feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `build`, `ci`.

Examples:

- `fix(tar): handle PAX path correctly`
- `docs: document overwrite behavior`
- `feat(cli)!: change default output directory`

## Checks (required)

Before pushing, ALWAYS run:

- `npm run lint`
- `npm run format`
- `npm run test-types`

Run `npm run format` even after creating/modifying Markdown files.

If you need to auto-fix issues:

- `npm run lint:fix`
- `npm run format`

---
> Source: [tiged/tiged](https://github.com/tiged/tiged) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
