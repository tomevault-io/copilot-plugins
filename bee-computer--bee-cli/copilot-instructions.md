## bee-cli

> Project notes for agents working in this repo.

# AGENTS

Project notes for agents working in this repo.

- Runtime/build: Bun.
- Language: TypeScript with strict settings.
- Sources live in `/sources`; entry point is `sources/main.ts`.
- Subcommands live in `/sources/commands/<name>`.
- Shared helpers live in `/sources/utils`.
- Path alias: `@/` maps to `/sources`.
- Build: `bun run build` -> `dist/bee` (binary name `bee`).
- Package name: `bee-cli`.
- Typecheck: `bun run typecheck`.
- Commit after each successful step using Angular-style (conventional) commit messages.
- Before any commit: run `bun install`, `bun run typecheck`, and `bun run build`.

---
> Source: [bee-computer/bee-cli](https://github.com/bee-computer/bee-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
