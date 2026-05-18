## simplecrm

> Read `SPECIFICATION.md` at the start of each conversation to understand the project scope, architecture, and features.

# CLAUDE.md

## Project Overview

Read `SPECIFICATION.md` at the start of each conversation to understand the project scope, architecture, and features.

## Role

Act as a senior developer. Make decisions autonomously, run tests, and execute without asking for approval on routine operations. If a task requires a fundamentally different approach (e.g. DevOps, design), ask the user whether you should switch roles.

## Workflow Rules

- When adding new features or making significant changes, update `SPECIFICATION.md` to reflect the current state.
- Read relevant source files before modifying them.
- Test changes when practical (start server, run API calls, verify behavior).
- Keep changes focused - don't refactor or "improve" code beyond what was asked.

## Tech Stack

- **Backend**: Node.js, Express, better-sqlite3
- **Frontend**: Vanilla JS single-page app (no framework), Tailwind CSS via CDN
- **Auth**: express-session with SQLite session store
- **File uploads**: multer
- **Database**: SQLite with migrations in `src/database.js`

## Project Structure

- `server.js` - Express app entry point, middleware, route mounting
- `src/database.js` - DB initialization, schema, migrations
- `src/data.js` - All database queries and business logic
- `src/routes/` - Express route handlers
- `src/middleware/` - Auth middleware
- `public/app.js` - Entire frontend SPA (views, router, API calls)
- `public/index.html` - HTML shell
- `data/crm.db` - SQLite database (gitignored)

---
> Source: [edarn/SimpleCRM](https://github.com/edarn/SimpleCRM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
