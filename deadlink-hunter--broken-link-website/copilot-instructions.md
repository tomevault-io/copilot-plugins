## broken-link-website

> Single source of truth for all AI coding agents (Cursor, Copilot, Claude Code, etc.).

# Agent Rules

Single source of truth for all AI coding agents (Cursor, Copilot, Claude Code, etc.).

- Remove all AI-generated comments.
- Avoid mega-files: split files larger than 300–400 lines into logical components or utilities.
- Never hardcode colors (e.g. `#2196F3`, `'red'`, `'white'`); use them from the theme.
- No hardcoded strings; use constants from the global constants folder.
- If a component has a lot of logic, extract it into a utility or custom hook.

Also follow the style guidelines in [CONTRIBUTING.md](./CONTRIBUTING.md).

---
> Source: [Deadlink-Hunter/Broken-Link-Website](https://github.com/Deadlink-Hunter/Broken-Link-Website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
