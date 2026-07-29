## wealth-tracker

> Wealth Tracker is a monorepo with a Svelte client and a Fastify + SQLite server.

# CLAUDE.md

Wealth Tracker is a monorepo with a Svelte client and a Fastify + SQLite server.
Keep this file concise and focused on rules Claude cannot reliably infer.

## Hard Rules
- MUST NOT edit generated build output in `server/public/**` or `client/dist/**`.
- MUST NOT hardcode secrets or keys. Use environment variables or `.env`.
- MUST register new API routes in `server/src/routes/index.ts`, or they will never run.
- MUST import any new Sequelize model in `server/src/index.ts` before `sequelize.sync()`.
- Schema changes MUST be backward-safe for SQLite. Add explicit migration logic if needed.

## Commands
- Install deps: `pnpm i` (preferred). Root scripts call `yarn`, so ensure Yarn is available.
- Dev server: `cd server && npm run dev` and `cd client && npm run dev`.
- Build: `yarn build` (client outputs to `server/public`, server outputs to `server/dist`).
- Start prod: `yarn start` or `cd server && npm run deploy` (pm2).
- Format: `yarn prettier` (root) or `cd client && npm run prettier`.
- Typecheck: `cd client && npm run check`.
- Tests: none currently. Say so if you did not run commands.

## Stack and Structure
- Client: Svelte 4 + Vite + Tailwind + Flowbite in `client/`.
- Server: Fastify + TypeScript + Sequelize + SQLite in `server/`.
- Client routes: `client/src/routes`. UI components: `client/src/components`.
- Server routes: `server/src/routes`. Controllers: `server/src/controllers`. Models: `server/src/models`.

## Runtime Config
- Server env: `PORT` (default 8888), `ALLOW_PASSWORD`, `CAN_BE_RESET`,
  `PEPPER_SECRET`, `OPENAI_API_KEY`, `OPENAI_BASE_URL`.
- Client env: `client/.env` uses `GOOGLE_ANALYTICS_KEY`.

## Style
- Prettier is authoritative: single quotes, no semicolons, 2-space indent, print width 100.

---
> Source: [nicejade/wealth-tracker](https://github.com/nicejade/wealth-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
