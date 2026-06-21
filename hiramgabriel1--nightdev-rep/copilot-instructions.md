## nightdev-rep

> - `pnpm dev` — start bot with `tsx watch` (auto-reload)

# AGENTS.md

## Commands

- `pnpm dev` — start bot with `tsx watch` (auto-reload)
- `pnpm build` — TypeScript compile to `dist/`
- `pnpm start` — run compiled output
- No lint or test scripts configured yet

## Architecture

- Single-process Telegram bot, entrypoint: `src/index.ts`
- `src/commands.ts` — `/start`, `/help` handlers
- `src/handlers.ts` — text message matching
- `src/logger.ts` — typed logger class with `LogLevel` enum

## Key conventions

- Uses `dotenv` — `src/index.ts` imports `'dotenv/config'` at top
- `TELEGRAM_BOT_TOKEN` required in `.env` (see `.env.example`)
- `node-telegram-bot-api` v1.1.0 uses ESM named exports, not namespace types
- TypeScript: `module: "Node16"`, `moduleResolution: "node16"`, `allowSyntheticDefaultImports: true`
- Import paths must include `.js` extension (ESNext module resolution)

## Git workflow

- Always commit changes after completing a task (do not wait for user to ask)
- Use Conventional Commits format: `type(scope): description`
  - Types: `feat`, `fix`, `docs`, `refactor`, `chore`, `test`, `style`, `perf`
  - Examples: `feat: add message logger`, `fix: resolve token loading order`
- Stage only files related to the change (`git add <files>`)
- Run `pnpm build` before committing to verify compilation
- Always `git push` after committing

---
> Source: [hiramgabriel1/nightdev_rep](https://github.com/hiramgabriel1/nightdev_rep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-21 -->
