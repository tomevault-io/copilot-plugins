## choir

> This file is a short orientation note for AI coding assistants working in this repository.

# CLAUDE.md

This file is a short orientation note for AI coding assistants working in this repository.

For project setup and operations, start with [README.md](README.md).
For codebase structure, see [docs/architecture.md](docs/architecture.md).

## Project Snapshot

CHOIR is a TypeScript Slack Bolt app for GitHub-backed knowledge workflows. It indexes markdown documentation, answers questions in Slack, and helps managers turn Slack conversations into GitHub document updates.

## Useful Commands

```bash
pnpm build
pnpm lint
pnpm test
pnpm verify
```

Use `pnpm dev:socket` for the usual local Socket Mode loop and `pnpm dev:oauth` for local HTTP/OAuth debugging.

## Code Map

- `app.ts`: application bootstrap and HTTP routes.
- `listeners/`: Slack event, action, and view handlers.
- `services/`: domain logic and integrations.
- `src/`: shared config, constants, utilities, and types.
- `docs/`: maintainer-facing notes.

## Working Notes

- Keep Slack payload parsing in listeners and put reusable domain behavior in `services/`.
- Prefer feature folders with a small `index.ts` registration file.
- Keep generated runtime output out of source control: `dist/`, `data/`, and `coverage/`.
- Do not commit real `.env` files or workspace data.

---
> Source: [wooogler/choir](https://github.com/wooogler/choir) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
