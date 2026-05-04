## effect-cf

> effect-cf is a pnpm + turbo monorepo providing typed Cloudflare clients (KV, Cache, Durable Objects) built on Effect.

# AGENTS

effect-cf is a pnpm + turbo monorepo providing typed Cloudflare clients (KV, Cache, Durable Objects) built on Effect.

## Quick start

- Install: pnpm install
- Build: pnpm build
- Test: pnpm test

## Common commands

- Format: pnpm format
- Format check: pnpm format:check
- Typecheck: pnpm typecheck
- Build single package: pnpm --filter @effect-cf/kv build
- Test single package: pnpm --filter @effect-cf/kv test
- Run single test: pnpm --filter @effect-cf/kv test -- --run -t "test name"

## Effect conventions

- Prefer namespace imports: import \* as Effect from "effect/Effect"
- Use combinators (flatMap/map/andThen) over Effect.gen
- Keep effects contextless: Effect.Effect<A, E, never>
- Use Effect.try/tryPromise for exceptions, Effect.mapError to normalize errors
- Define errors via Data.TaggedError in packages/core

## Client patterns

- Create clients via createXClient, provide via XLayer
- get/match return null; getOrFail/matchOrFail raise NotFoundError
- Decode unknown with core helper, encode before JSON.stringify
- Use Schema.Schema<A, I> types; keep schemas in callers
- Export tag alias (e.g. KvClient) from Context.Tag

## Error handling

- BindingsError for missing bindings, ValidationError for parse/encode failures
- ResponseError for non-2xx HTTP responses (include status/body)
- TransportError for wrapped thrown causes
- Never throw; return failed Effect values

## TypeScript and formatting

- ESM only; strict mode with exactOptionalPropertyTypes and noUncheckedIndexedAccess
- Prettier: 100 char width, 2 spaces, semicolons, no trailing commas
- Use type-only imports for types: import type { T } from "./types"
- Prefer readonly on public types, omit optional fields instead of undefined

## File structure

- packages/\*/src/index.ts re-exports public APIs
- Tests: src/\*.test.ts, use @effect-cf/testing Miniflare helpers
- Type prefixes: Kv*, Cache*, DurableObjects\*
- Never edit dist/ (tsc output)

## Do / Don't

- Do run pnpm typecheck before finalizing changes
- Do run pnpm --filter @effect-cf/<pkg> test for package validation
- Do use Effect.runPromise for success paths, Effect.runPromiseExit + Cause.failureOption for errors
- Don't commit secrets; check .prettierrc.json for formatting rules
- Don't introduce implicit retries or caching in clients
- Don't throw; return failed Effect values
- Don't edit dist/ or commit node_modules/

---
> Source: [jbt95/effect-cf](https://github.com/jbt95/effect-cf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
