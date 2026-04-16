## sms-cli

> Were moving to cloudflare workers: see "https://developers.cloudflare.com/workers/llms-full.txt"

Were moving to cloudflare workers: see "https://developers.cloudflare.com/workers/llms-full.txt"
Always use `bun` over `npm`
# SMS CLI

Two-component SMS management system: `sms-server` (Hono HTTP + SQLite) and `sms` (CLI).

## Tech Stack
- Bun runtime, Hono framework, `bun:sqlite`
- `commander` + `chalk` for CLI
- `bun build --compile` for standalone binaries

## Commands
```
bun run dev:server           # Run server with watch
bun run dev:cli -- <cmd>     # Run CLI
bun run build                # Compile sms-server + sms binaries
bun test                     # Run all tests
bun test:unit                # Unit tests only
bun test:integration         # Integration tests only
```

## Development
- Use `bun` not node/npm
- Use Hono for HTTP routing (CF Workers portable)
- Use `bun:sqlite` not better-sqlite3
- Use `Bun.file` over node:fs readFile/writeFile
- Bun auto-loads `.env`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alexbruf) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
