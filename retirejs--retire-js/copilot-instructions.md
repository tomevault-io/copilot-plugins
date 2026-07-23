## retire-js

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository structure

This is a monorepo with two main parts:

- **`node/`** — The CLI tool and core library (TypeScript, published to npm as `retire`). Source in `node/src/`, compiled output in `node/lib/`.
- **`repository/`** — The vulnerability database and tooling to manage it.

## Node CLI (node/)

```bash
cd node
npm install
npm run build       # compile TypeScript → lib/
npm run test        # run jest test suite (node/spec/tests/*.spec.ts)
npm run check       # lint + typecheck
npm run lint        # eslint --fix
npm run typecheck   # tsc --noEmit
```

Run a single test file:
```bash
cd node
npx jest spec/tests/contentscan.spec.ts
```

## Vulnerability repository (repository/)

The source of truth is `jsrepository-master.json`. The other `jsrepository*.json` files are derived outputs and must be kept in sync.

After editing `jsrepository-master.json`, regenerate all derived files:
```bash
cd repository
node convertToVersioned
```

Validate that all derived files are in sync and the format is correct:
```bash
cd repository
node validate
```

Test that detection regexes actually match the real library files (downloads from CDN, cached in `repository/tmp/`):
```bash
cd repository
node test-detection.js
node test-detection.js <library-name>   # test a single library
```

**Before submitting a PR:** always run `validate` and `test-detection.js`.

## Repository format

Entries in `jsrepository-master.json` use `§§version§§` as a placeholder for a version-capturing regex group. Required fields per vulnerability:

- `severity` — align with CVE/GHSA severity when available
- `cwe` — array of CWE IDs
- `identifiers` — at least one of: `CVE`, `githubID`, `pr`, `issue`

Version ranges: omit `atOrAbove` if no lower bound is known; use `"999.0.0"` for `below` when no upper bound is known.

Do **not** edit `jsrepository.json` directly — it is generated from master. Do **not** submit to `npmrepository.json` (deprecated).

## Core detection logic (node/src/)

- `retire.ts` — core scanning functions: `scanUri`, `scanFileName`, `scanFileContent`, `check`
- `scanner.ts` — filesystem walker, orchestrates scanning files and npm packages
- `deepscan.ts` — AST-based detection for libraries that embed version strings in non-obvious ways
- `repo.ts` — loads and validates the repository (uses zod schema)
- `cli.ts` — commander-based CLI entry point
- `reporters/` — output formatters (console, JSON, CycloneDX SBOM)

## Bumping the version number

When bumping the version number of node/package.json, also update the version number in:
- node/lib/retire.js
- chrome/extension/manifest.json
- chrome/extension-no-func/manifest.json

---
> Source: [RetireJS/retire.js](https://github.com/RetireJS/retire.js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
