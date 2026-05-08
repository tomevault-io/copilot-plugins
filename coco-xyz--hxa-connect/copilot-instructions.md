## hxa-connect

> Development guidelines for hxa-connect.

# CLAUDE.md

Development guidelines for hxa-connect.

## Project Conventions

- **TypeScript** — All source in `src/`, compiled to `dist/` via `tsc`
- **ESM only** — `"type": "module"` in package.json
- **Node.js 20+** — Minimum runtime version
- **Conventional commits** — `feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `test:`
- **Secrets in `.env` only** — Never commit secrets
- **English for code** — Comments, commit messages, PR descriptions, and documentation in English

## Release Process

When releasing a new version, **all three files** must be updated in the same commit:

1. **`package.json`** — Bump `version` field
2. **`package-lock.json`** — Run `npm install` after bumping package.json to sync the lock file
3. **`CHANGELOG.md`** — Add new version entry following [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) format

Version bump commit message: `chore: bump version to X.Y.Z`

After merge, create a GitHub Release with tag `vX.Y.Z` from the merge commit.

## Architecture

HXA-Connect is a **bot-to-bot communication hub** — lightweight, self-hostable messaging infrastructure for AI bots.

- `src/index.ts` — Main entry point (Express + WebSocket server)
- `src/routes.ts` — HTTP API routes
- `src/ws.ts` — WebSocket connection management
- `src/auth.ts` — Authentication and token management
- `src/db.ts` — Database abstraction (SQLite / PostgreSQL)
- `src/db/` — Database migrations and queries
- `src/types.ts` — Shared type definitions
- `src/webhook.ts` — Webhook delivery

## Testing

```bash
npm test
```

---
> Source: [coco-xyz/hxa-connect](https://github.com/coco-xyz/hxa-connect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
