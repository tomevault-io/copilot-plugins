## cmux-tui

> This project uses Bun as the primary runtime and build tool.

# Development Instructions

This project uses Bun as the primary runtime and build tool.

## Default to Bun

- Use `bun <file>` instead of `node <file>` or `ts-node <file>`
- Use `bun test` instead of `jest` or `vitest`
- Use `bun build <file.html|file.ts|file.css>` instead of `webpack` or `esbuild`
- Use `bun install` instead of `npm install` or `yarn install` or `pnpm install`
- Use `bun run <script>` instead of `npm run <script>` or `yarn run <script>` or `pnpm run <script>`
- Use `bunx <package> <command>` instead of `npx <package> <command>`
- Bun automatically loads `.env`, so don't use `dotenv`

## APIs

- `Bun.serve()` supports WebSockets, HTTPS, and routes. Don't use `express`
- `bun:sqlite` for SQLite. Don't use `better-sqlite3`
- `Bun.redis` for Redis. Don't use `ioredis`
- `Bun.sql` for Postgres. Don't use `pg` or `postgres.js`
- `WebSocket` is built-in. Don't use `ws`
- Prefer `Bun.file` over `node:fs`'s `readFile`/`writeFile`
- Use `Bun.$` for shell commands instead of `execa`

## Running the Project

```bash
# Development with hot reload
bun run dev

# Production run
bun run start

# Tests
bun test
```

## More Information

For complete Bun API documentation, see the official Bun documentation at https://bun.sh/docs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/W4M-ai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
