## pi-kagi-api

> Simple guidance for coding agents working in this repository.

# AGENTS.md

Simple guidance for coding agents working in this repository.

## Repository setup

- Requirements: Node.js + npm
- Install dependencies:

```bash
npm install
```

## Local validation

This package is a Pi extension with entry point `index.ts`.

Run type checking:

```bash
npm run typecheck
```

Check what would be published:

```bash
npm pack --dry-run
npm publish --dry-run
```

Manual check with local package:

```bash
KAGI_API_KEY="your-api-key" pi -e .
```

Then run Pi and invoke `kagi_search` or `kagi_extract`.

## Code map

- `index.ts` — extension entry point, config/API key management, Kagi Search/Extract API calls, tool + command registration
- `README.md` — user-facing install/configuration docs
- `CHANGELOG.md` — release history
- `package.json` — package metadata and Pi extension declaration

## Commit format

Prefer:

- Imperative mood
- Sentence case
- No prefix like `feat:` / `fix:` / `chore:`

Examples:

- `Add Kagi Extract tool`
- `Document API key setup`
- `Improve search result formatting`

Keep commits focused (one logical change per commit).

## Release notes

- Package name: `@mjakl/pi-kagi-api`
- For user-visible changes, update `CHANGELOG.md`.
- For npm releases, bump the version (`npm version patch|minor|major`) and publish.

---
> Source: [mjakl/pi-kagi-api](https://github.com/mjakl/pi-kagi-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
