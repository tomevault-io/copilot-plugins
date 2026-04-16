## cf-neocities

> This project is a web application built with [Hono](https://hono.dev/) and [Cloudflare Workers](https://workers.cloudflare.com/), utilizing [Durable Objects](https://developers.cloudflare.com/durable-objects/) (with SQLite storage) for state management. It emulates classic "Neocities" style features like a visit counter, a BBS (Bulletin Board System), and a personal diary.

# CF-Neocities Project Context

## Project Overview
This project is a web application built with [Hono](https://hono.dev/) and [Cloudflare Workers](https://workers.cloudflare.com/), utilizing [Durable Objects](https://developers.cloudflare.com/durable-objects/) (with SQLite storage) for state management. It emulates classic "Neocities" style features like a visit counter, a BBS (Bulletin Board System), and a personal diary.

## Architecture

### Core Stack
- **Framework:** Hono (v4+)
- **Runtime:** Cloudflare Workers
- **Storage:** Durable Objects with SQLite API
- **Language:** TypeScript
- **Testing:** Vitest with `@cloudflare/vitest-pool-workers`

### Components (Durable Objects)
The application relies on three main Durable Objects, defined in `wrangler.jsonc` and `src/`:

1.  **Counter (`src/Counter.ts`):**
    - Manages a global visit count.
    - Handles "Kiriban" (milestone) detection (e.g., 100, 1111).
    - Generates SVG images for the counter display.

2.  **BBS (`src/BBS.ts`):**
    - A simple bulletin board system.
    - Stores posts with ID, name, body, date, delete key, and IP hash.
    - Supports pagination and logical deletion.

3.  **Diary (`src/Diary.ts`):**
    - A blog/diary system.
    - Stores entries by date (YYYY-MM-DD) as the primary key.
    - Supports CRUD operations protected by Basic Authentication.

### Templating
- Uses a custom build step using `micro-template`.
- HTML files in `templates/` are compiled into a single JavaScript module `src/templates.js` via `npm run build:templates`.
- Helper functions (`formatDateForDisplay`, `nl2br`, etc.) are passed to templates at runtime.

### Static Assets
- Public assets are served from `public/`.
- SVG digits for the counter are processed from `public/digits/` and embedded via `scripts/embed-digits.js` into `src/digits.js`.

## Key Files & Directories

- **`src/index.ts`**: Main entry point. Defines Hono routes, middleware (CSRF protection, Basic Auth), and bindings.
- **`src/BBS.ts`, `src/Counter.ts`, `src/Diary.ts`**: Durable Object class definitions.
- **`templates/`**: HTML templates. Edited here, then compiled.
- **`wrangler.jsonc`**: Cloudflare Workers configuration, including DO migrations and bindings.
- **`scripts/embed-digits.js`**: Script to bundle SVG assets.

## Development Workflow

### Commands
- **Install Dependencies:** `npm install`
- **Start Local Dev Server:** `npm run dev` (uses Miniflare/workerd)
- **Build Templates:** `npm run build:templates` (Run this after modifying `.html` files in `templates/`)
- **Run Tests:** `npm run test`
- **Deploy:** `npm run deploy`
- **Type Generation:** `npm run cf-typegen` (Generates `worker-configuration.d.ts`)

### Configuration
- **Secrets:**
    - `DIARY_AUTH_USERNAME`: Username for Diary Basic Auth (default: 'admin').
    - `DIARY_AUTH_PASSWORD`: Password for Diary Basic Auth (default: 'password').
- **Bindings:** Defined in `wrangler.jsonc` and typed in `src/index.ts`.

## Coding Conventions
- **Routing:** Use Hono's chainable route definitions (`app.get`, `app.post`).
- **Database:** Use `this.ctx.storage.sql` within Durable Objects for all data persistence. Use parameterized queries to prevent SQL injection.
- **Security:**
    - **CSRF:** The `requireXRequestedWith` middleware enforces `x-requested-with: XMLHttpRequest` on non-GET methods.
    - **Auth:** `diaryBasicAuth` middleware protects sensitive Diary routes.
- **Dates:** Dates are generally stored as ISO8601 strings or `YYYY-MM-DD` strings.

## Notes for AI
- **Template Changes:** If you modify files in `templates/`, you MUST run `npm run build:templates` for changes to take effect in the application code.
- **State Management:** Logic involving state (posts, counts, diary entries) must be implemented within the corresponding Durable Object class, not in `index.ts`.
- **Imports:** Note that `src/templates.js` and `src/digits.js` are build artifacts. Do not edit them directly.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cho45) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
