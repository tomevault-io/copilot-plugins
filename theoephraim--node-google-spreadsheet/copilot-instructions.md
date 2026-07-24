## node-google-spreadsheet

> `google-spreadsheet` — a Google Sheets API wrapper for Node.js (published as `google-spreadsheet` on npm).

# CLAUDE.md

## Project Overview
`google-spreadsheet` — a Google Sheets API wrapper for Node.js (published as `google-spreadsheet` on npm).

Goal of the project is to provide a simplified, more ergonomic interface compared to Google's official SDKs.

## Tech Stack
- TypeScript (strict mode), ESM module
- Built with `tsdown` (outputs CJS + ESM to `dist/`)
- Tested with `vitest`
- Linted with `eslint`
- Uses `bun` as package manager
- Env vars (for testing) managed with `varlock`
- Versioning/releases managed with `@varlock/bumpy`

## Common Commands
- `bun test` — run tests (vitest in watch mode)
- `bun run test <file>` — run specific test file (e.g., `bun run test src/test/data-operations.test.ts`)
- `bun run test:ci` — run tests once
- `bun run build` — build with tsdown
- `bun run lint` — run eslint
- `bun run lint:fix` — run eslint with auto-fix
- `bun changeset` — create a changeset for version bumps

## Project Structure
- `src/` — source code
  - `src/index.ts` — package entry point
  - `src/lib/` — internal utilities and helpers
    - `src/lib/GoogleSpreadsheet.ts` — main document class
    - `src/lib/GoogleSpreadsheetWorksheet.ts` — worksheet class
    - `src/lib/GoogleSpreadsheetRow.ts` — row class
    - `src/lib/GoogleSpreadsheetCell.ts` — cell class
  - `src/test/` — test files (`*.test.ts`)
- `docs/` — docsify documentation site

## Workflow
- Always run `bun run lint` before committing to catch lint errors.
- Always add a changeset (`bun changeset`) for new features and bug fixes.

## Testing
- Tests hit real Google APIs against test documents — they are integration tests, not mocked.
- Tests run sequentially (`fileParallelism: false` in vitest config).
- Test files: `src/test/*.test.ts`

---
> Source: [theoephraim/node-google-spreadsheet](https://github.com/theoephraim/node-google-spreadsheet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
