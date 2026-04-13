## yearn-insights

> Guidance for Claude Code when working with this repository.

# CLAUDE.md

Guidance for Claude Code when working with this repository.

## Quick Start

```bash
bun install
cp .env.example .env     # Add ETH_RPC_URL and per-chain RPCs
bun run db:migrate       # Create SQLite tables
bun run seed             # Fetch all vault data (~2 min)
bun run dev:api          # Terminal 1: Hono API on :3456
bun run dev:dashboard    # Terminal 2: Vite React on :5173
```

## Commands

```bash
# Development
bun run dev:api          # Hono API on :3456 (--watch)
bun run dev:dashboard    # Vite React dashboard on :5173 (proxies /api → :3456)

# Core data fetching (or use `bun run seed` for all four)
bun run fetch:kong       # V2/V3 vaults from Kong GraphQL API
bun run fetch:defillama  # DefiLlama protocol TVL snapshots
bun run fetch:curation   # Morpho Blue API + Turtle Club on-chain reads
bun run fetch:v1         # V1 legacy vaults on-chain (Ethereum only)

# Additional data (run after seed)
bun run fetch:reports    # Vault harvest reports (~5 min)
bun run fetch:prices     # Weekly historical asset prices from DefiLlama (~10 min)
bun run fetch:depositors # Depositor data from Kong transfers (Ethereum only)

# Enrichment (run after fetching)
bun run scripts/fetch-v2-fees.ts    # Read actual V2 fee rates on-chain
bun run scripts/reprice-reports.ts  # Reprice reports using cached prices + snapshot fallback

# Analysis CLI (JSON default, --table for human-readable)
bun run audit                              # TVL summary (JSON)
bun run audit tvl --table                  # TVL table
bun run audit overlaps                     # Strategy→vault overlaps
bun run audit vault 0x...                  # Single vault detail
bun run audit fees --table                 # Fee summary
bun run audit depositors                   # Depositor concentration
bun run audit -- --chain=1 --min-tvl=1000000  # Filter flags
bun run audit help                         # Show all subcommands
bun run scripts/detect-overlaps.ts         # Find overlaps on-chain

# Database & types
bun run db:migrate       # Apply migrations
bun run typecheck        # tsc --noEmit across all packages
bun run lint             # Biome check (lint + format)
bun run lint:fix         # Auto-fix lint/format issues
bun run format           # Format all files
```

## Populating Data

Full pipeline for accurate metrics:

```bash
bun run seed                            # 1. Core vault data
bun run seed:extra                      # 2-5. All enrichment steps below in one command
```

Or run enrichment steps individually:

```bash
bun run fetch:reports                   # 2. Harvest reports
bun run scripts/fetch-v2-fees.ts        # 3. On-chain V2 fee rates
bun run fetch:prices                    # 4. Historical asset prices
bun run scripts/reprice-reports.ts      # 5. Reprice reports with cached prices
```

## API Endpoints

Base: `https://yearn-metrics.fly.dev` (production) or `http://localhost:3456` (local)

```
GET /health                                  # Health check

GET /api/tvl                                 # TVL summary (totals, by chain, by category)
GET /api/tvl/vaults?chainId=1&category=v2    # Per-vault list (filterable)
GET /api/tvl/overlap                         # Vault→vault overlap details

GET /api/comparison                          # Our TVL vs DefiLlama

GET /api/fees?since=1709251200               # Fee summary (optional since= unix ts)
GET /api/fees/vaults?since=1709251200        # Per-vault fee breakdown
GET /api/fees/history?interval=weekly        # Fee history (weekly|monthly)

GET /api/analysis/dead                       # Dead TVL (no reports in 365d)
GET /api/analysis/retired                    # Retired vault TVL
GET /api/analysis/sticky                     # Sticky TVL analysis
GET /api/analysis/depositors/:address        # Depositor concentration (?chainId=1)
```

## Architecture

```
packages/shared  →  packages/db  →  packages/api  →  packages/dashboard
                                 ↗
              scripts/ (data fetchers)
```

### packages/shared
Types, constants, and utilities shared across all packages:
- **types.ts** — `VaultCategory`, `TvlSummary`, `KongVault`, `DefillamaComparison`
- **constants.ts** — `CHAIN_NAMES`, `KONG_API_URL`, `V1_VAULTS`, `IGNORED_VAULTS`
- **pricing.ts** — `fetchCurrentPrices()`, `CHAIN_PREFIXES`, `DefiLlamaPriceProvider`
- **time.ts** — `toMondayNoon()`, `weeklyTimestamps()`, `WEEK_SECONDS`, `YEAR_SECONDS`
- **collections.ts** — `groupBy()`, `toMap()`
- **curation.ts** — Morpho owner/curator addresses, Turtle Club vaults, factory configs
- **strategy-overlaps.ts** — `STRATEGY_OVERLAP_REGISTRY`, `CROSS_CHAIN_OVERLAP_REGISTRY`

### packages/db
SQLite via Drizzle ORM + `bun:sqlite`. DB at `packages/db/yearn-tvl.db`.

Tables: `vaults`, `vaultSnapshots`, `strategies`, `strategyDebts`, `feeConfigs`, `strategyReports`, `defillamaSnapshots`, `depositors`, `assetPrices`

Import: `import { db, vaults, ... } from "@yearn-tvl/db"`

### packages/api
Hono REST API on port 3456. Route groups:
- `/api/tvl` — TVL aggregation with overlap deduction (auto + registry + cross-chain)
- `/api/comparison` — Our TVL vs DefiLlama per chain and category
- `/api/fees` — Fee revenue from harvests, time-weighted management fees
- `/api/analysis` — Dead TVL, retired TVL, depositor concentration

### packages/dashboard
React 19 + Vite + Recharts. 5 tabs: Overview, Comparison, Fees, Analysis, Vaults. Uses `useFetch<T>` hook with `VITE_API_URL` prefix (empty in local dev, full URL in production).

### scripts/
| Script | Source | Output |
|--------|--------|--------|
| `fetch-kong.ts` | Kong GraphQL API | vaults, snapshots, strategies, debts, fees |
| `fetch-defillama.ts` | DefiLlama protocol API | defillamaSnapshots |
| `fetch-curation.ts` | Morpho Blue API + on-chain reads | curation vaults |
| `fetch-v1-vaults.ts` | On-chain reads (Ethereum) | V1 vaults + snapshots |
| `fetch-reports.ts` | Kong vaultReports query | strategyReports |
| `fetch-depositors.ts` | Kong transfers query | depositors (Ethereum) |
| `fetch-historical-prices.ts` | DefiLlama historical prices | assetPrices (weekly) |
| `fetch-v2-fees.ts` | On-chain reads | V2 fee rate corrections |
| `reprice-reports.ts` | assetPrices cache | repriced strategyReports |
| `detect-overlaps.ts` | On-chain balanceOf reads | overlap candidates (stdout) |
| `refresh-retired.ts` | On-chain reads + DL prices | retired vault TVL updates |
| `audit.ts` | DB reads | interactive TUI |

## Key Concepts

**Vault categories**: `v1` (legacy, Ethereum only), `v2` (apiVersion 0.4.x), `v3` (v3=true), `curation` (Morpho/Turtle Club, not in Kong)

**Vault types (V3)**: `1` = allocator (deposits into strategies), `2` = strategy (receives allocations). Showing only allocators OR only strategies avoids double-counting.

**Double-counting**: Capital flows vault → strategy → another vault. Three detection methods:
1. *Auto*: strategy address = known vault address on same chain
2. *Registry*: `STRATEGY_OVERLAP_REGISTRY` for intermediary contracts
3. *Cross-chain*: `CROSS_CHAIN_OVERLAP_REGISTRY` for retired vaults whose capital migrated chains

**TVL formula**: `totalTvl = active + retired - autoOverlap - registryOverlap - crossChainOverlap`

**Fee calculation**:
- Performance: `gainUsd × (performanceFee / 10000)` per harvest
- Management: time-weighted `totalAssets × weeklyAssetPrice × rate × duration` (falls back to latest TVL if no price data)
- Rates in basis points (1000 = 10%)

**Report pricing priority**: Kong gainUsd → cached weekly asset price (±1 week) → vault snapshot price → $500K cap

**DL comparison**: V1+V2+V3 ↔ `yearn-finance`, Curation ↔ `yearn-curating`. Both deduct overlap per-category.

**Chains**: Ethereum (1), Optimism (10), Polygon (137), Fantom (250), Base (8453), Arbitrum (42161), Gnosis (100), Katana (747474), Hyperliquid (999), Berachain (80094), Sonic (146)

## Deployment

| Component | Host | URL |
|-----------|------|-----|
| API | Fly.io | https://yearn-metrics.fly.dev |
| Dashboard | Vercel | https://yearn-metrics-dashboard.vercel.app |

### Update production DB

```bash
bun run seed && bun run fetch:reports && bun run scripts/fetch-v2-fees.ts && bun run fetch:prices && bun run scripts/reprice-reports.ts
fly ssh console -C "rm /data/yearn-tvl.db /data/yearn-tvl.db-shm /data/yearn-tvl.db-wal"
fly ssh sftp shell  # put packages/db/yearn-tvl.db /data/yearn-tvl.db
fly machine restart <machine-id>
```

### Deploy API / Dashboard

```bash
fly deploy                               # API → Fly.io
cd packages/dashboard && vercel --prod   # Dashboard → Vercel
```

## Environment

`.env` at project root (never committed):
- `VITE_API_URL` — Production API URL for dashboard. Not needed locally.
- `DB_PATH` — Override SQLite path. Default: `packages/db/yearn-tvl.db`. Set to `/data/yearn-tvl.db` on Fly.
- `ETH_RPC_URL` / `RPC_URI_FOR_{chainId}` — Per-chain RPCs for on-chain reads.

## Conventions

- All imports use `.js` extensions (ESM with TypeScript)
- Prefer `const` over `let`; use `map`/`filter`/`reduce` over imperative loops
- Workspace packages referenced as `workspace:*` in package.json
- Scripts export a named async function and run via `if (import.meta.main)`
- Shared utilities go in `packages/shared/src/` (pricing, time, collections, types)
- DB path resolved relative to `packages/db/` via `import.meta.url`, not CWD
- 9 vaults in `IGNORED_VAULTS` excluded from all calculations

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yearn)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yearn)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
