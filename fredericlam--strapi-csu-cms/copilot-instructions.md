## strapi-csu-cms

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run develop    # Start with auto-reload (development)
npm run start      # Start without auto-reload (production)
npm run build      # Build the admin panel
npm run seed:example  # Populate database with example content
```

No test or lint commands are configured.

## Node Version

This project requires **Node 22** (Strapi 5.40.0 supports Node 18–22). A `.nvmrc` is included:

```bash
nvm use   # switches to Node 22
```

## Environment Setup

Copy `.env.example` to `.env` and replace placeholder values:

```
HOST=0.0.0.0
PORT=1337
APP_KEYS=<two comma-separated random strings>
API_TOKEN_SALT=<random string>
ADMIN_JWT_SECRET=<random string>
TRANSFER_TOKEN_SALT=<random string>
JWT_SECRET=<random string>
```

## Architecture

This is a **Strapi 5 headless CMS** for a blog platform. The default database is SQLite (`.tmp/data.db`). MySQL and PostgreSQL are supported via environment variables in `config/database.js`.

### Content Types (`src/api/`)

| Type | Kind | Key fields |
|------|------|-----------|
| `article` | Collection | title, slug, cover, author (relation), category (relation), blocks (dynamiczone) |
| `author` | Collection | name, avatar, email |
| `category` | Collection | name, slug |
| `global` | Single | siteName, favicon, defaultSeo (component) |
| `about` | Single | title, blocks (dynamiczone) |

### Shared Components (`src/components/shared/`)

- `media` — single file attachment
- `quote` — title + body text
- `rich-text` — markdown content
- `slider` — multiple images
- `seo` — meta title, description, share image

### Dynamic Zones

Both `article` and `about` use a `blocks` dynamiczone that accepts: `shared.media`, `shared.quote`, `shared.rich-text`, `shared.slider`.

### Permissions

Public (unauthenticated) read access is configured for articles, authors, categories, global, and about. The seed script (`scripts/seed.js`) handles setting these permissions programmatically.

### Config

- `config/api.js` — REST defaults: limit 25, max 100
- `config/database.js` — SQLite default; MySQL/PostgreSQL via env vars
- `config/server.js` — host `0.0.0.0`, port `1337`
- `config/middlewares.js` — standard Strapi middleware stack

### Seed Data

`/data/data.json` contains sample categories, authors, articles, and global settings. Sample images are in `/data/uploads/`. Run `npm run seed:example` to populate.

### Generated Types

`types/generated/` contains auto-generated TypeScript definitions — do not edit manually.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fredericlam)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/fredericlam)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
