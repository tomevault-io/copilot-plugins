## scraper

> This is a Node.js web scraper service that fetches US visa bulletin data from the State Department website (travel.state.gov), stores bulletins in a DynamoDB database via Cyclic.sh, and sends email notifications when a target bulletin becomes available.

# CLAUDE.md

## Project Overview

This is a Node.js web scraper service that fetches US visa bulletin data from the State Department website (travel.state.gov), stores bulletins in a DynamoDB database via Cyclic.sh, and sends email notifications when a target bulletin becomes available.

## Architecture

Single-file Express application (`index.js`, ~100 lines). No subdirectories or modular structure.

### Core Flow
1. `GET /` triggers the scrape
2. `getBulletins()` fetches HTML from travel.state.gov
3. Cheerio parses `.current` elements for bulletin text and links
4. `save()` persists bulletins to DynamoDB via `@cyclic.sh/dynamodb`
5. `getAll()` retrieves all stored bulletins
6. If a target bulletin is found (via `lodash _.find()`), `send()` emails a notification through Gmail SMTP

### Key Functions (all in `index.js`)
- `getBulletins()` — HTTP GET to State Department, returns raw HTML
- `save(list)` — Writes bulletin array to DynamoDB `bulletins` collection
- `getAll()` — Lists and fetches all items from `bulletins` collection
- `send(message)` — Sends email via nodemailer/Gmail SMTP

## Tech Stack

| Component       | Library                       |
|-----------------|-------------------------------|
| Web framework   | Express 4.x                   |
| HTTP client     | Axios 1.x                     |
| HTML parsing    | Cheerio 1.x                   |
| Database        | @cyclic.sh/dynamodb (DynamoDB)|
| Email           | Nodemailer + SMTP transport   |
| Utilities       | Lodash                        |

## Development

### Prerequisites
- Node.js
- npm

### Install dependencies
```
npm install
```

### Run locally
```
npx nodemon index.js
```
The server listens on port 3000. Visit `http://localhost:3000/` to trigger a scrape.

### No test suite
There are no tests configured. `npm test` exits with an error placeholder.

## Code Conventions

- **Module system:** CommonJS (`require()` / `module.exports`)
- **Style:** No linter or formatter configured (no ESLint, Prettier)
- **Variables:** camelCase naming
- **Async patterns:** Mix of `async/await` and raw Promises
- **No TypeScript** — plain JavaScript only

## File Structure

```
/
├── index.js          # Entire application (entry point, routes, scraping, DB, email)
├── package.json      # Dependencies and metadata
├── package-lock.json # Locked dependency versions
└── .gitignore        # Excludes node_modules/
```

## Known Issues

- Credentials are hardcoded in `index.js` rather than sourced from environment variables
- `getBulletins()` silently resolves to `undefined` on HTTP errors
- `save()` uses `.map()` with async callbacks without awaiting all promises
- `getAll()` uses an undeclared loop variable (`for (item of ...)` — missing `const`/`let`)
- Target bulletin key (`'July2024'`) is hardcoded
- No error handling middleware on the Express app
- No structured logging

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gong4494) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
