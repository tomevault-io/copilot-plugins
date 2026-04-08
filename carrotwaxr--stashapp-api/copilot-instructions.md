## stashapp-api

> TypeScript npm package providing a typed GraphQL client for Stash media organizer servers.

# stashapp-api

TypeScript npm package providing a typed GraphQL client for Stash media organizer servers.

## Current State (v0.4.0 -> v1.0.0 migration in progress)

**v0.4.0 (legacy):** graphql-codegen + 38 hand-written `.graphql` operations, `StashApp` singleton class, fixed field selections.

**v1.0.0 (target):** GenQL-generated typed client with Prisma-style `{ field: true }` field selection, full API coverage (74 queries + 129 mutations), `StashClient` wrapper class.

## Development

```bash
# GenQL generate + compile (v1.0)
npm run build          # runs generate + tsc

# Schema snapshot (for diffing across Stash versions)
npm run schema:snapshot

# Full refresh
npm run refresh        # schema snapshot + build
```

Requires `.env` with `STASH_ENDPOINT` and `STASH_API_KEY` pointing to a running Stash server (10.0.0.4:6969).

## Architecture

- `src/client.ts` - `StashClient` class (auth, query/mutation proxies, raw escape hatch)
- `src/index.ts` - Public API re-exports (StashClient + all GenQL types)
- `src/generated/` - GenQL output (gitignored, regenerated from schema introspection)
- `genql.config.js` - GenQL codegen configuration (endpoint, scalar type mappings)

## Key Dependencies

- `@genql/runtime` - GenQL client runtime
- `@genql/cli` (dev) - Code generation from GraphQL introspection
- `typescript` - Compilation

## Publishing

```bash
npm run publish:patch  # or :minor or :major
```

Runs `prepublishOnly` hook (full build) before publishing.

## Consumers

- `stashapp-cli` (`/home/carrot/code/stashapp-cli`) - CLI tool, only consumer

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/carrotwaxr)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/carrotwaxr)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
