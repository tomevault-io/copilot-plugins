## devstash

> A developer knowledge hub for snippets, commands, prompts, notes, files, images, links and custom types.

# DevStash

A developer knowledge hub for snippets, commands, prompts, notes, files, images, links and custom types.

## Context Files

Read the following to get the full context of the project:

- @context/project-overview.md
- @context/coding-standards.md
- @context/ai-interaction.md
- @context/current-feature.md

## Neon Database (MCP)

The DevStash Neon project does **not** appear in `mcp__neon__list_projects`. Always query the database using the connection string from `.env` via the `@neondatabase/serverless` driver:

```js
const { neon } = require('@neondatabase/serverless');
const sql = neon('<DATABASE_URL from .env>');
```

| Branch      | Endpoint                   | Connection string source |
|-------------|----------------------------|--------------------------|
| development | `ep-cool-base-adkxkun8`    | `.env`                   |
| production  | `ep-flat-truth-ad0gmg29`   | `.env.production`        |

- **Database:** `neondb` (both branches)
- **Default for all dev work:** use `.env` (development branch)

> Never use `mcp__neon__run_sql` with another project's ID for DevStash queries — it will hit the wrong database.

## Commands

- **Dev server**: `npm run dev` (runs on http://localhost:3000)
- **Build**: `npm run build`
- **Production server**: `npm run start`
- **Lint**: `npm run lint`

**IMPORTANT:** Do not add Claude to any commit messages.

**IMPORTANT:** When completing a feature please add information to the bottom of the Histroy section explaning what was done in this feature

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ameetj85) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
