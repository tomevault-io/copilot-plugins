## emdedd

> npm run build        # Compile TS → dist/

# Commands
npm run build        # Compile TS → dist/
npm test             # Run tests (requires build first)
npm run build && npm test  # Full verify

# Architecture
Single-file CLI tool: `src/index.ts` → compiled to `dist/src/index.js`
- Parses markdown for `<!-- ts-embed: -->` / `<!-- cs-embed: -->` markers
- Extracts TS symbols via TypeScript Compiler API, C# via regex
- Replaces marker content with current source declarations

# Key Files
- src/index.ts       — All logic (parsing, extraction, CLI)
- test/run.ts        — Integration tests (no framework, uses child_process)
- test/fixtures/     — Test data (sample.ts, input.md, expected.md)

# Code Style
- ESM ("type": "module" in package.json)
- TypeScript strict mode
- No test framework — plain Node.js assertions

# Gotchas
- Tests run from `dist/test/run.js` but reference source fixtures via `../../test/fixtures/`
- Must `npm run build` before `npm test` — tests execute compiled output
- `files` in package.json scoped to `dist/src` — test code not published
- CI publishes to npm on `v*` tags — requires NPM_TOKEN secret in GitHub repo settings or Trusted Publisher role for GitHub Actions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shlomiassaf) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
