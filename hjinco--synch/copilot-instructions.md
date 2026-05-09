## synch

> Synch is an end-to-end encrypted Obsidian Sync alternative. The repository is a pnpm workspace with:

## Project Overview

Synch is an end-to-end encrypted Obsidian Sync alternative. The repository is a pnpm workspace with:

- `apps/api`: Cloudflare Workers API, Hono, Drizzle, Better Auth.
- `apps/obsidian-plugin`: Obsidian plugin client.
- `apps/www`: Astro website.
- `packages/*`: shared workspace packages.

Prioritize preserving end-to-end encryption guarantees, vault safety, and compatibility with Obsidian plugin behavior.

## Package Manager

Use `pnpm`. Do not use `npm` or `yarn`.

## Common Commands

From the repository root:

```sh
pnpm -C apps/api test:unit
pnpm -C apps/api test:integration
pnpm -C apps/api test:e2e:smoke
pnpm -C apps/api typecheck

pnpm -C apps/obsidian-plugin test
pnpm -C apps/obsidian-plugin typecheck
pnpm -C apps/obsidian-plugin build

pnpm -C apps/www build
```

---
> Source: [hjinco/synch](https://github.com/hjinco/synch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
