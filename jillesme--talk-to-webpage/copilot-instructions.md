## talk-to-webpage

> This is a [Flue](https://flueframework.com) project: agents are TypeScript functions.

# AGENTS.md

This is a [Flue](https://flueframework.com) project: agents are TypeScript functions.

## Layout

- `src/agents/` — agent modules. A module whose first line is the `'use agent'` directive exports agents: every exported capitalized function is one, and the function name is its durable identity.
- `src/app.ts` — the route map; every route is mounted here explicitly.
- `src/cloudflare.ts` — Worker-level exports and non-HTTP handlers.
- `wrangler.jsonc` — Worker config; every agent needs a Durable Object migration entry.

## Commands

- `pnpm dev` — start the Cloudflare dev server. Use HTTP/`curl` against mounted application routes; the Cloudflare-native model does not run through `flue run`.
- `pnpm deploy` — build and deploy the Worker.
- `pnpm check:types` — typecheck.
- `pnpm exec flue docs search <query>` — search the Flue docs from the terminal (then `pnpm exec flue docs read <path>`).
- `pnpm exec flue add` — list blueprints for adding channels, sandboxes, and databases.

---
> Source: [jillesme/talk-to-webpage](https://github.com/jillesme/talk-to-webpage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
