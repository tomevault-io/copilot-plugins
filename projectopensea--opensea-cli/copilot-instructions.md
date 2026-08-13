## opensea-cli

> TypeScript CLI and programmatic SDK for the OpenSea API.

# cli — Agent Conventions

TypeScript CLI and programmatic SDK for the OpenSea API.

## Quick commands

```bash
cd packages/cli
pnpm run build
pnpm run test
pnpm run type-check
pnpm run lint
pnpm run format
```

## Responsibilities

- `opensea` CLI: Commander-driven commands for all OpenSea API v2 domains.
- `OpenSeaCLI` / `OpenSeaClient` SDK: typed, camelCase wrappers over the REST API.
- Output formatting: JSON (default), table, and server-side TOON.

## Rules

1. **Never hand-roll API request/response types**. Import from `@opensea/api-types` via `src/types/api.ts`. Run `pnpm --filter @opensea/api-types run update-spec && build` before adding endpoints.
2. **API path check**. `pnpm check-api-paths` fails if source code references an `/api/v2/...` URL not in `packages/api-types/opensea-api.json`.
3. **CLI commands are thin**. Parse args, call `OpenSeaClient`, format output with `formatOutput()`. Keep business logic in the client.
4. **One file per domain in `src/commands/`**. Register new commands in `src/commands/index.ts` and wire in `src/cli.ts`.
5. **SDK classes in `src/sdk.ts`**. Add domain classes as `readonly` properties on `OpenSeaCLI` for programmatic consumers.
6. **Auth session invariants**. Persisted tokens require access token, refresh token, wallet address, ISO expiration, scopes, and `oauth`/`siwe` method. Normalize only `0x`-prefixed EVM addresses to lowercase; preserve Solana base58 case.
7. **Exit codes**. `0` success, `1` API error, `2` auth error, `3` rate limited.

## Conventions

- ESM-only, `.js` import extensions, `verbatimModuleSyntax` on.
- Biome: double quotes, 2-space indent, trailing commas, 80-char line width.
- Native `fetch` only; no axios/got/node-fetch.
- Cursor-based pagination with `--next <cursor>`; never offset-based.

---
> Source: [ProjectOpenSea/opensea-cli](https://github.com/ProjectOpenSea/opensea-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
