## ghsecret

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development

```bash
npm run build        # tsc → dist/
npm run dev          # tsc --watch
npm run lint         # tsc --noEmit (type check only)
npm test             # vitest run
npm run test:watch   # vitest (watch mode)
```

After building, test the CLI directly: `node dist/cli.js --help`

For global dev usage: `npm link` → then use `ghsecret` anywhere.

## Architecture

React + Ink terminal UI app with meow for CLI argument parsing.

**Data flow:** `cli.tsx` parses argv into `AppConfig` → `render(<App />)` → App routes to either `<Interactive />` (wizard) or `<Push />` (execution). Push has its own state machine: `checking → show-warnings|confirm|mixed-pick → pushing → done`.

**Two rendering modes:**
- `Help.tsx` uses `chalk` + `console.log` (runs before Ink, exits immediately)
- Everything else uses Ink React components with `useInput` for keyboard handling

**gh CLI calls:** All in `src/utils/gh.ts`. User-input-touching calls MUST use `execFileSync("gh", argsArray)` — never `execSync` with string concatenation (shell injection prevention). Only `checkDeps()` uses `execSync` with hardcoded strings.

**Precedence system:** GitHub resolves secrets/variables as Environment (3) > Repository (2) > Organization (1). `checkPrecedence()` in `gh.ts` scans other levels and warns when a push would be shadowed or would shadow another level.

**Exit codes:** All error paths must call `exit(new Error(...))` via Ink's `useApp()` hook — never bare `exit()` or `process.exit(1)` from components. The `waitUntilExit()` chain in `cli.tsx` translates this to process exit codes.

## Key Constraints

- **ESM-only** (`"type": "module"`) — all imports require `.js` extensions
- **TypeScript strict mode** — no `any`, no `@ts-ignore`
- **VERSION** is read from `package.json` at runtime via `createRequire` — never hardcode version strings
- **`ink-select-input`** and **`ink-spinner`** are installed but unused — available for future use

## Testing

Tests are in `src/__tests__/`. Only the env parser is unit-tested (Ink components require a TTY).

```bash
npx vitest run src/__tests__/env-parser.test.ts   # single test file
```

Test pattern: `beforeAll` creates temp `.env` files in `os.tmpdir()`, `afterAll` cleans up.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/barisdemirhan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
