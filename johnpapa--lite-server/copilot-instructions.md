## lite-server

> **lite-server** is a lightweight development-only Node.js server that wraps [BrowserSync](https://www.browsersync.io/) to serve SPAs. It provides browser auto-refresh, CSS injection via sockets, and HTML5 History API fallback routing so deep links work during development.

# AI-Ready Repo — Agent Guide

## Project Overview

**lite-server** is a lightweight development-only Node.js server that wraps [BrowserSync](https://www.browsersync.io/) to serve SPAs. It provides browser auto-refresh, CSS injection via sockets, and HTML5 History API fallback routing so deep links work during development.

## Repository Structure

```
lite-server/
├── bin/lite-server             # CLI entry point
├── lib/
│   ├── config-defaults.js      # Default BrowserSync configuration
│   └── lite-server.js          # Core server logic — config loading, merging, BrowserSync init
├── index.js                    # Public API entry point (exports server + defaults)
├── test/
│   ├── config-defaults.spec.js # Unit tests for default config
│   └── lite-server.spec.js     # Unit tests for core server logic
├── .eslintrc                   # ESLint configuration (strict rule set)
├── package.json                # npm metadata, dependencies, scripts
├── CHANGELOG.md                # Version history
├── README.md                   # Usage docs, configuration guide, contributing
└── LICENSE                     # MIT
```

## Tech Stack

| Area | Tool |
|------|------|
| Runtime | Node.js |
| Package manager | npm |
| Core dependency | BrowserSync |
| Linting | ESLint (strict config in `.eslintrc`) |
| Testing | Mocha + Istanbul (coverage) |
| Mocking | Mockery + Sinon |
| Assertions | Node.js built-in `assert` |

## Build & Run

There is no build step. This is a pure JavaScript package.

```bash
# Install dependencies
npm install

# Run linting + tests with coverage
npm test
```

The `npm test` command runs:

1. `eslint *.js lib/*.js` — lint all source files
2. `istanbul cover _mocha -- -R spec` — run Mocha tests with Istanbul coverage

## How It Works

1. **CLI** (`bin/lite-server`) — sets process title and calls the server function
2. **Server** (`lib/lite-server.js`) — parses CLI args with `minimist`, loads the user's `bs-config.{js|json}` overrides, deep-merges with defaults using `lodash.merge`, and starts BrowserSync
3. **Defaults** (`lib/config-defaults.js`) — provides sensible BrowserSync defaults: file watching for `*.{html,htm,css,js}`, `connect-logger` middleware, and `connect-history-api-fallback` middleware
4. **Public API** (`index.js`) — exports `{ server, defaults }` for programmatic use

## Key Patterns

- **Config merging** — uses `lodash.merge()` to deep-merge user overrides with defaults
- **Middleware array** — uses `lodash.compact()` to remove `null` entries (allows users to disable specific middleware by index)
- **Function configs** — `bs-config.js` can export a function that receives the BrowserSync instance
- **Strict linting** — `.eslintrc` enforces `'use strict'`, single quotes, 2-space indent, camelCase, max line length 100
- **Test isolation** — uses `mockery` to mock `require()` calls; each test enables/disables mockery in `beforeEach`/`afterEach`

## Adding a Feature

1. Create a feature branch: `git checkout -b feat/my-feature`
2. Modify or add source files in `lib/`
3. Add or update tests in `test/` using the Mockery + Sinon pattern
4. Run `npm test` to ensure lint + tests pass
5. Update `CHANGELOG.md` with your changes
6. Submit a pull request against `main`

## Common Pitfalls

- **Mockery cleanup** — always call `mockery.deregisterAll()` and `mockery.disable()` in `afterEach` to avoid test pollution
- **`useCleanCache: true`** — required when using mockery so the module cache doesn't bleed between tests
- **Middleware overrides** — `lodash.merge` merges objects by key, so array-like objects `{ 0: null }` set specific middleware slots to `null`
- **No TypeScript** — this is plain JavaScript with `'use strict'` directives; don't add TS files
- **No build step** — don't add bundlers or compilers; the package ships raw JS

---
> Source: [johnpapa/lite-server](https://github.com/johnpapa/lite-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
