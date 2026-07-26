## plugma

> Preferences regarding code style

- Your notes folder is `.cursor/notes` (at the root of the workspace or projects in monorepos) is yours to use. Only you will ever read the contents in there. Use it to store plans, task lists, notes, learnings, and important reminders for you.
- When your response contains the code of a markdown file, code blocks inside said markdown file MUST use "~~~"
- when importing node native modules, like fs or url, always prepend them with 'node:', like in 'node:fs' and 'node:url'
- Always document exported functions, classes, types, etc with a robust TSDoc comment. Also document complex functions, even if not exported.
- Use "for ... of" loops instead `.forEach`.
- to prevent issues with commands being aliased, always use `command BUILTIN` for built-in commands, like `command cat` or `command ls`.

---
> Source: [gavinmcfarland/plugma](https://github.com/gavinmcfarland/plugma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
