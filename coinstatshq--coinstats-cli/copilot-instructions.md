## coinstats-cli

> CoinStats CLI for the CoinStats Public API, designed for AI agents.

# AGENTS.md

CoinStats CLI for the CoinStats Public API, designed for AI agents.

## Commands

```bash
npm install
npm run generate:client
npm run build
npm test
node dist/index.js schema --pretty
```

## Layout

```text
src/
├── index.ts               # CLI entry point
├── cli.ts                 # Commander setup and command execution
├── core/
│   ├── config.ts          # Config file, env resolution, base URL
│   ├── openapi.ts         # OpenAPI snapshot loading and parameter helpers
│   ├── output.ts          # JSON/table/CSV formatting
│   └── sdk.ts             # Generated SDK invocation wrapper
├── generated/             # Auto-generated CoinStats SDK from OpenAPI
└── registry/
    └── commands.ts        # One-to-one command mapping for every public operation
```

## Rules

- Keep command coverage at 100% of the public OpenAPI snapshot.
- Add or update entries in `src/registry/commands.ts` when the public API changes.
- Re-run `npm run generate:client` whenever `openapi/coinstats-public-api.json` changes.
- Keep mutating operations behind `--yes`.
- Prefer JSON request bodies via `--body` or `--body-file` for POST/PATCH operations.

## Testing

- `npm test` must pass.
- `npm run build` must pass.
- `npm run check:coverage` must report that every public operation is mapped.
- Mock `fetch` in unit tests. Do not hit the live CoinStats API in automated tests.

## Gotchas

- The generator currently requires `./openapi/...` rather than `openapi/...` in the CLI script input.
- The published Mintlify OpenAPI snapshot currently exposes 51 paths and 52 operations.
- Vitest and ESLint currently target supported Node LTS versions. Avoid Node 23 in CI.

---
> Source: [CoinStatsHQ/coinstats-cli](https://github.com/CoinStatsHQ/coinstats-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
