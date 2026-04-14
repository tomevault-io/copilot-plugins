## t2000

> Token data architecture — canonical sources for token metadata


# Token Data Architecture

## Canonical sources (single source of truth)

| Data | Source | Location |
|------|--------|----------|
| Swap-supported token names → coin types | `TOKEN_MAP` | `packages/sdk/src/protocols/cetus-swap.ts` |
| NAVI lending asset metadata | `SUPPORTED_ASSETS` | `packages/sdk/src/constants.ts` |
| Coin type → symbol/decimals (RPC fallback) | `KNOWN_COINS` | `packages/engine/src/sui-rpc.ts` |
| NAVI MCP decimal corrections | `NAVI_NEWER_POOL_SYMBOLS` | `packages/engine/src/navi-transforms.ts` |

## Rules

1. **Never create a new token map.** Import from the canonical source above.
2. **When adding a new token**, update `TOKEN_MAP` in the SDK. Everything downstream (system prompt, swap resolution) derives from it automatically.
3. **Decimal precision**: USDSUI, USDe, suiUSDT are 6 decimals. NAVI MCP incorrectly treats them as 9 (see `UPSTREAM_WORKAROUNDS.md`). The correction lives in `navi-transforms.ts` ONLY.
4. **Frontend `useBalance.ts`** has its own `KNOWN_COINS` — this should eventually be replaced with an import from the SDK, but for now keep it in sync manually and document any additions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mission69b)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mission69b)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
