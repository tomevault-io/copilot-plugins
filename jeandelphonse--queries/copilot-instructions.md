## queries

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is an e-commerce data utilities project that provides query functions for a SQLite database. The project uses TypeScript with ES2022 modules (`"type": "module"` in `package.json`).

## Development Commands

```bash
# Install dependencies
npm run setup          # npm install + generates .claude/settings.local.json from template

# Run the main cron job script
node --import tsx/esm src/main.ts

# Run the Agent SDK demo
npm run sdk            # runs sdk.ts via tsx
```

## Environment Variables

`SLACK_WEBHOOK_URL` must be set for `main.ts` to send alerts. Add it to `.env` or the system environment.

## Architecture

### Cron job (`src/main.ts`)

Runs once per day. Opens `ecommerce.db`, ensures the schema exists, then queries for pending orders older than 3 days and sends a Slack alert per order to `#order-alerts` via `src/slack.ts`.

### Query modules (`src/queries/`)

All database query functions live here — this is enforced by a hook (see below). Functions return `Promise<any>` or `Promise<any[]>` and use parameterized queries via `db.get()` / `db.all()`. Column names must match `src/schema.ts` (e.g. `id`, `created_at`, `customer_id` — not `order_id`, `order_date`).

### Agent SDK (`sdk.ts`, `hooks/query_hook.js`)

Uses `@anthropic-ai/claude-agent-sdk` (`query()`) to run Claude Code as a subprocess. `query_hook.js` uses this to review proposed query additions for duplication against existing queries — currently short-circuited (`process.exit(0)` at top of `main()`).

### Hooks (`.claude/settings.local.json`)

Generated from `.claude/settings.example.json` by `npm run setup` (replaces `$PWD`).

| Event | Matcher | Hook | Behavior |
|---|---|---|---|
| PreToolUse | `Write\|Edit\|MultiEdit` | `hooks/query_hook.js` | Blocks writes to `src/queries/` if new query duplicates existing functionality (currently disabled) |
| PreToolUse | `Read` | `hooks/read_hook.js` | Blocks reads of `.env` files |
| PostToolUse | `Write\|Edit\|MultiEdit` | prettier + `hooks/tsc.js` | Auto-formats file, then type-checks entire project via TypeScript compiler API |

Exit code 2 from any hook = hard block with the hook's stderr as feedback.

### Database Schema

Defined in `src/schema.ts` via `createSchema()`. Key tables and their primary key / FK conventions:
- `customers` — `id`, FK'd from `orders.customer_id`
- `orders` — `id`, `status` (`pending`|`processing`|`shipped`|`delivered`|`cancelled`|`refunded`), `created_at`
- `order_items` — `id`, FK `order_id`, `product_id`
- `products` — `id`, FK `category_id`
- `inventory` — composite unique `(product_id, warehouse_id)`

## Critical Guidance

- All database queries must be written in `./src/queries/`
- The `package-lock.json` resolved URL for `@anthropic-ai/claude-agent-sdk` may point to a private Artifactory registry. If `npm install` fails with E401, update the `resolved` field for that package to `https://registry.npmjs.org/@anthropic-ai/claude-agent-sdk/-/claude-agent-sdk-0.1.5.tgz`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JeanDelphonse)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/JeanDelphonse)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
