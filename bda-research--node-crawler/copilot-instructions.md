## node-crawler

> This file provides guidance to the AI agent when working with code in this repository.

# AGENTS.md

This file provides guidance to the AI agent when working with code in this repository.

## Build & Test

Tests run against compiled output in `dist/`, not source. Always build before testing:

```sh
npm run build   # tsup → dist/index.js + dist/index.cjs
npm test        # NODE_ENV=test ava (runs test/*.js, excludes test/*test.js)
```

CI pipeline: `npx eslint && npm run build && npm test`

## Testing

- **ava** with **nock** for HTTP mocking. No real network calls.
- Test files are plain `.js` (not `.ts`) and import from `../dist/index.js`.
- Ava config excludes `test/*test.js` — new test files should NOT use the `.test.js` suffix (e.g. `callback.js`, not `callback.test.js`).
- Tests use a custom `testCb` wrapper from `test/lib/avaTestCb.js` for callback-based async. Call `t.end()` to signal completion.
- Default test timeout is 20s.

## Code Style

- TypeScript strict mode, ESM throughout (`"type": "module"`).
- ESLint enforced on `src/**/*.ts`: double quotes, semicolons required, `no-console` is an error, unused vars prefixed with `_` are allowed.
- Target: ES2020. Module resolution: bundler.

## Architecture Notes

- `got` is lazy-loaded on first use via dynamic `import("got")` — do not add a top-level static import for it.
- `cheerio` is the default HTML parser (exposed as `res.$`). jsdom is not supported.
- Rate limiter uses a cluster of independent limiters. `rateLimit` is global-only — modify per-limiter via `crawler.setLimiter()`, not per-request options.
- The `callback` pattern requires calling `done()` to signal task completion and release the slot.

---
> Source: [bda-research/node-crawler](https://github.com/bda-research/node-crawler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
