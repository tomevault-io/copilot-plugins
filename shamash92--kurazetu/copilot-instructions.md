## kurazetu

> These instructions apply to the React web application under `src/ui/`.

# React web instructions

These instructions apply to the React web application under `src/ui/`.

## Read first

- Repository-wide agent rules: [`../../AGENTS.md`](../../AGENTS.md)
- JavaScript and TypeScript code style:
  [`../../.claude/code-style.md`](../../.claude/code-style.md)
- React and React Native testing principles:
  [`../../.claude/testing-principles.md`](../../.claude/testing-principles.md)
- Frontend setup: [`../../docs/tutorials/setup.md`](../../docs/tutorials/setup.md)

## Working rules

- Use `pnpm` and the scripts in `package.json`; do not substitute another
  package manager.
- If a code file under `src/ui/` changes, run `pnpm lint`. Run `pnpm test` when
  the change affects behavior covered by Jest tests.
- Visible UI changes require the repository's
  [Impeccable UI QA guidance](../../.claude/design/impeccable.md).
- Prefer the configured `@/...` alias for imports outside the current folder.

---
> Source: [shamash92/KuraZetu](https://github.com/shamash92/KuraZetu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
