## opensea-sdk

> TypeScript SDK for buying, selling, and managing NFTs and tokens on OpenSea. Supports ethers and viem providers.

# sdk — Agent Conventions

TypeScript SDK for buying, selling, and managing NFTs and tokens on OpenSea. Supports ethers and viem providers.

## Quick commands

```bash
cd packages/sdk
pnpm run build
pnpm run test
pnpm run test:integration  # requires OPENSEA_API_KEY
pnpm run check-types
pnpm run lint
```

## Responsibilities

- Provide `OpenSeaSDK` (ethers) and `OpenSeaViemSDK` (viem) entry points.
- Provide the Stream API client at the `@opensea/sdk/stream` subpath.
- Camelize API responses and expose typed helpers for orders, fulfillment, assets, and wallet auth.
- Keep the `Chain` enum in sync with `ChainIdentifier` from `@opensea/api-types`.

## Rules

1. **Never hand-roll API request/response types**. Import from `@opensea/api-types` (or re-export through `src/api/types.ts`) using canonical schema names.
2. **Chain enum sync is compile-time enforced**. Adding a `ChainIdentifier` without a matching `Chain` value or payment-token case fails `pnpm check-types`. Update `scripts/chain-data.json` at the monorepo root and run `pnpm sync-chains` when adding chains.
3. **Dual provider support**. Changes to `BaseOpenSeaSDK` affect both ethers and viem paths; update both provider adapters if provider-specific logic changes.
4. **OAuth token contract**. `OpenSeaOAuth` requests `offline_access`; refresh responses may omit rotation — keep the previous refresh token. The top-level `wallet` JWT claim is wallet identity; `sub` is an account id.
5. **No secret leakage**. API keys live in `OpenSeaAPIConfig.apiKey`; never log them.
6. **Auth scopes are coupled to the spec in both directions**. `src/scopes.ts` asserts at compile time that `OPENSEA_SCOPES` matches `AuthScope` from `@opensea/api-types`, so a new scope has to land in the spec and in the constant in the same commit, and that commit cannot pass the `Mirror layout` gate until api-types publishes. Read [Spec changes and release order](../../AGENTS.md#spec-changes-and-release-order) before starting.
7. **Stream client is subpath-only and dependency-free**. `src/stream/` is exported from `./stream` and never from `src/index.ts` — `EventType`, `Trait`, `TraitOfferEvent`, and `CollectionOfferEvent` exist in both surfaces with different shapes. `src/stream/transport/` stays internal so a non-Phoenix Stream v2 can replace it without a breaking release; client code uses `StreamTransport`, never `PhoenixChannelsTransport`. The built entry resolves to six local files with zero external requires, so check the require graph before adding an import. Live tests are in `test/integration/stream.spec.ts` and need real network access, see `test/README-integration.md`.

## Conventions

- CommonJS (`"type": "commonjs"`) for broad consumer support.
- Node 22+ is the floor. The stream client relies on a global `WebSocket`.
- `viem` is an optional peer dependency; main entry uses ethers.
- Prefer `string` for decimal `Amount` values.

---
> Source: [ProjectOpenSea/opensea-sdk](https://github.com/ProjectOpenSea/opensea-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
