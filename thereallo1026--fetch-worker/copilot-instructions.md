## fetch-worker

> This project is a Cloudflare Worker that acts as a fetch API proxy/debugger. It accepts JSON in fetch format, executes the actual request, and returns the response.

# Fetch Worker Project Overview

This project is a Cloudflare Worker that acts as a fetch API proxy/debugger. It accepts JSON in fetch format, executes the actual request, and returns the response.

## Structure
- [src/index.ts](mdc:src/index.ts) - Main entry point for the worker
- [src/handlers/](mdc:src/handlers/) - Request handlers
- [src/utils/](mdc:src/utils/) - Utility functions
- [wrangler.jsonc](mdc:wrangler.jsonc) - Cloudflare Workers configuration

## Development
- Uses Bun as the JavaScript runtime
- Run `bun run dev` to start a local development server
- Run `bun run deploy` to deploy to Cloudflare

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Thereallo1026) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
