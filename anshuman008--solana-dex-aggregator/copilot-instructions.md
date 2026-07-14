## solana-dex-aggregator

> solroute is a Rust DEX aggregator for Solana. 6 pure DEX crates parse on-chain account data and compute swap outputs through a unified `Market` trait. An aggregator crate loads all pools from RPC, finds multi-hop routes, and provides pricing/caching. An engine crate runs a persistent HTTP API with live gRPC streaming. No external APIs -- all data is on-chain.

# Repository Guidelines

## Project Overview

solroute is a Rust DEX aggregator for Solana. 6 pure DEX crates parse on-chain account data and compute swap outputs through a unified `Market` trait. An aggregator crate loads all pools from RPC, finds multi-hop routes, and provides pricing/caching. An engine crate runs a persistent HTTP API with live gRPC streaming. No external APIs -- all data is on-chain.

## Architecture

```
solroute-core          Market trait, shared types, constants
    ^
    |
    +-- raydium-amm-v4    Constant product AMM
    +-- raydium-clmm      Concentrated liquidity (Q64.64 sqrt_price)
    +-- meteora-damm      Dynamic AMM V1 (constant product + stable) and V2 (sqrt_price)
    +-- meteora-dlmm      Dynamic liquidity bins
    +-- pumpfun-amm       Bonding curve (virtual reserves)

solroute-aggregator    Pool loading, routing, pricing, caching, CLI
solroute-engine        HTTP API, AccountStore, PoolRegistry, cold_start, streaming
solroute        Root crate: re-exports all DEX crates
```

No DEX crate imports another DEX crate.

### Data Flow (pure DEX crates)

```
Raw account bytes --BorshDeserialize--> Pool model struct
                                            |
                                            v
                                     DexMarket::new(pool, address, balances)
                                            |
                                            v
                                     impl Market { ... }
                                       +-- calculate_output()
                                       +-- calculate_price_impact()
                                       +-- current_price()
                                       +-- is_active()
```

## Key Directories

```
solroute/
+-- bin/
|   +-- engine.rs                       # Engine binary: immediate serve, background cold start
+-- Cargo.toml                          # Workspace root
+-- src/lib.rs                          # Root crate: re-exports all DEX crates
+-- crates/
|   +-- core/src/
|   |   +-- traits.rs                   # Market trait, AccountDataProvider, calculate_output_live
|   |   +-- constants.rs                # WSOL, USDC, USDT, quote_priority, infer_mint_decimals
|   +-- raydium-amm-v4/src/lib.rs       # RaydiumAMMV4 + RaydiumAmmV4Market
|   +-- raydium-clmm/src/
|   |   +-- lib.rs                      # RaydiumCLMMPool + RaydiumClmmMarket
|   |   +-- tick_arrays.rs              # Tick array bitmap + PDA derivation
|   +-- meteora-damm/src/
|   |   +-- lib.rs                      # V1 MeteoraDAMMMarket + V2 MeteoraDAMMV2Market
|   |   +-- models.rs                   # Pool models for V1, V2
|   |   +-- utils.rs                    # derive_token_vault_address
|   +-- meteora-dlmm/src/lib.rs         # MeteoraDLMMPool + MeteoraDlmmMarket
|   +-- pumpfun-amm/src/lib.rs          # PumpfunAmmPool + PumpfunAmmMarket
|   +-- aggregator/src/                 # Pool loading, routing, pricing, caching, CLI
|   |   +-- loader.rs                   # Async RPC pool loading (all DEXs)
|   |   +-- cache.rs                    # Disk cache + PDA extraction
|   |   +-- router.rs                   # Multi-hop routing (live data via AccountDataProvider)
|   |   +-- price.rs                    # SOL/USD on-chain via CLMM sqrt_price
|   |   +-- pool_index.rs              # In-memory token-pair graph
|   |   +-- cli.rs                      # Progress bars + interactive REPL
|   |   +-- main.rs                     # CLI binary (solroute-cli)
|   +-- engine/src/                     # Persistent service (library only, binary in bin/)
|   |   +-- account_store.rs            # DashMap store, implements AccountDataProvider
|   |   +-- pool_registry.rs            # Swappable validation, vault-to-pool reverse index
|   |   +-- cold_start.rs              # Background: vault fetch, tick arrays, bin arrays, bitmap exts
|   |   +-- streaming.rs                # Yellowstone gRPC: live updates + vault re-validation
|   |   +-- api.rs                      # Axum HTTP: /quote, /price, /health
+-- tests/
|   +-- helpers/mod.rs                  # Shared Geyser test helpers
|   +-- trade_stream.rs                 # Live DEX swap streaming via Yellowstone gRPC
|   +-- creation_stream.rs              # Live token + pool creation streaming
|   +-- pool_financials.rs              # Live pool update streaming
|   +-- validate_prices.rs              # Price validation across DEXs
```

## Development Commands

```bash
cargo check                        # Type-check all workspace crates
cargo build                        # Build all workspace crates
cargo test --workspace --lib       # Run unit tests
cargo build --release --bin solroute-engine  # Build engine binary
cargo build --release -p solroute-aggregator  # Build aggregator CLI

# Run engine (persistent service with HTTP API)
RPC_URL="https://..." cargo run --release --bin solroute-engine

# Run aggregator CLI (interactive REPL)
RPC_URL="https://..." cargo run --release -p solroute-aggregator

# Engine API
curl http://localhost:8080/health
curl "http://localhost:8080/quote?inputMint=SOL&outputMint=6p6xgHyF7AeE6TZkSmFsko444wqoP15icUSqi2jfGiPN&amount=100000000&maxHops=2"
curl "http://localhost:8080/price?mint=SOL"
```

### Environment Variables

| Variable | Default | Description |
|---|---|---|
| `RPC_URL` | `https://api.mainnet-beta.solana.com` | Solana RPC endpoint |
| `CACHE_PATH` | `pools.cache` | Pool cache file location |
| `CACHE_MAX_AGE` | `3600` | Max cache age (seconds) before RPC reload |
| `GEYSER_ENDPOINT` | (none) | Yellowstone gRPC endpoint for live streaming |
| `GEYSER_TOKEN` | (none) | Yellowstone gRPC auth token |
| `PORT` | `8080` | solroute engine HTTP API port |
| `PRIVATE_KEY` | (none) | Base58 keypair in `.env` (never committed) |

### Engine Startup

The engine starts serving HTTP immediately after cache load (~6s). No blocking vault fetch.

```
1. Load cache              ~6s   pools.cache -> PoolIndex
2. validate_from_cache     instant  uses market.financials() (cached vault balances)
3. Start HTTP server       instant  /quote works immediately
4. gRPC streaming          background  live account updates + vault re-validation
5. Vault fetch             background  4M+ accounts, 100 concurrent batches
6. SOL/USD price refresh   background  every 15s
```

`validate_from_cache()` uses the vault balances already embedded in the deserialized market objects. No RPC calls. Once the background vault fetch completes, `validate_all(store)` re-validates with fresh on-chain data.

### Cold Start Auxiliary Fetches

After vault loading completes in the background:

1. **DLMM bitmap extensions** -- single GPA for `dataSize=12488` accounts
2. **CLMM tick arrays** -- PDA-derived from each pool's `tick_array_bitmap`, fetched via `getMultipleAccounts` (top 10k pools by vault balance, ~6 tick arrays each)
3. **DLMM bin arrays** -- PDA-derived from each pool's `active_id`, fetched via `getMultipleAccounts`

### Pool Loading

Fetches ALL pools from all 6 DEXs using `getProgramAccounts` with discriminator + dataSize filters only (no mint filter). ~2M pools total. Vault balances cached per-pool during loading (used for instant startup validation).

### Pool Cache

Saved to `pools.cache` (~1.6 GB) after first load. Subsequent startups load in ~6s vs ~4min from RPC. Uses bincode-serialized `CachedPool` enum with per-DEX pool variants. `CachedPool` also serves as the source for PDA extraction (`extract_clmm_tick_pdas`, `extract_dlmm_bin_pda`) during cold start.

### Swappable Validation

Route discovery gates on pool status and liquidity:

| DEX | Swappable when |
|---|---|
| Pumpfun AMM | always (no status field) |
| Raydium AMM V4 | vaults funded |
| All others | `is_active()` AND vaults funded |

`calculate_output_live()` reads live pool state (sqrt_price, active_id, liquidity) and vault balances from the `AccountStore` via the `AccountDataProvider` trait on every quote request. Streaming triggers `on_vault_update()` to re-validate swappable status when Token Program or Token-2022 vault balances change.

### REPL Commands

```
solroute> price SOL                    # SOL price in USD
solroute> price <mint>                 # Token price in SOL + USD
solroute> route SOL <mint> 1.0         # Find best routes
solroute> stats                        # Pool counts, memory, uptime
solroute> exit
```

## Code Conventions

### Market Trait

```rust
pub trait Market: Send + Sync {
    fn is_active(&self) -> bool { true }
    fn metadata(&self) -> Result<PoolMetadata, GenericError>;
    fn financials(&self) -> Result<PoolFinancials, GenericError>;
    fn calculate_output(&self, amount_in: u64, direction: SwapDirection) -> Result<u64, GenericError>;
    fn calculate_output_live(&self, amount_in: u64, direction: SwapDirection,
        pool_data: Option<&[u8]>, quote_vault_balance: u64, base_vault_balance: u64,
    ) -> Result<u64, GenericError> { self.calculate_output(amount_in, direction) }
    fn calculate_price_impact(&self, amount_in: u64, direction: SwapDirection) -> Result<u64, GenericError>;
    fn current_price(&self) -> Result<f64, GenericError>;
}

pub trait AccountDataProvider: Send + Sync {
    fn pool_account_data(&self, pubkey: &Pubkey) -> Option<Vec<u8>>;
    fn token_balance(&self, vault_pubkey: &Pubkey) -> u64;
}
```

### Pool Status & `is_active()`

Each DEX has its own on-chain status semantics. `is_active()` overrides check the raw status field:

| DEX | Status field | Active condition | Source |
|---|---|---|---|
| Raydium V4 | `status: u64` | `status == 6` | Raydium AMM status enum |
| Raydium CLMM | `status: u8` (bitfield) | `status & (1 << 4) == 0` | Bit 4 = disable swap |
| Meteora DAMM V1 | `enabled: bool` | `enabled == true` | Direct boolean |
| Meteora DAMM V2 | `pool_status: u8` | `pool_status == 0` | `PoolStatus::Enable = 0` |
| Meteora DLMM | `status: u8` | `status == 0` | `PairStatus::Enabled = 0` |
| Pumpfun AMM | (none) | default `true` | No on-chain status field |

### Error Handling

`type GenericError = Box<dyn Error + Send + Sync>` -- string errors via `.into()`. No `thiserror` or `anyhow`.

### Constants
- DEX-specific program IDs in each DEX crate
- Shared constants (WSOL, USDC, USDT, TOKEN_PROGRAM, etc.) in `solroute_core`
- Quote currency ordering: `solroute_core::quote_priority()`
- Token decimal inference: `solroute_core::infer_mint_decimals()`

### Pool Discovery Filters

| DEX | Program ID | data_size | Anchor Discriminator |
|---|---|---|---|
| Raydium V4 | `675kPX...` | 752 | None |
| Raydium CLMM | `CAMMC...` | 1544 | `[247,237,227,245,215,195,222,70]` |
| Meteora DAMM V1 | `Eo7Wj...` | 944 | `[241,154,109,4,17,177,109,188]` |
| Meteora DAMM V2 | `cpamd...` | 1112 | `[241,154,109,4,17,177,109,188]` |
| Meteora DLMM | `LBUZKh...` | 904 | `[33,11,49,98,181,101,177,13]` |
| Pumpfun AMM | `pAMMB...` | N/A | `[241,154,109,4,17,177,109,188]` |

## Important Files

| File | What it is |
|---|---|
| `bin/engine.rs` | Engine binary: instant startup, background vault fetch, 15s SOL/USD refresh |
| `crates/engine/src/account_store.rs` | DashMap store, implements `AccountDataProvider` for live routing |
| `crates/engine/src/pool_registry.rs` | Swappable validation (cached `Arc<HashSet>`, vault-to-pool reverse index) |
| `crates/engine/src/cold_start.rs` | Background: vault fetch (100 concurrent), tick arrays, bin arrays, bitmap exts |
| `crates/engine/src/streaming.rs` | Yellowstone gRPC: live updates + vault re-validation (Token + Token-2022) |
| `crates/engine/src/api.rs` | Axum HTTP: GET /quote, GET /price, GET /health |
| `crates/aggregator/src/router.rs` | Multi-hop routing (live data, pre-resolved mints, cached swappable set) |
| `crates/aggregator/src/loader.rs` | RPC pool loading (discriminator + dataSize filters) |
| `crates/aggregator/src/cache.rs` | Disk cache + PDA extraction helpers |
| `crates/core/src/traits.rs` | Market trait, AccountDataProvider, calculate_output_live |
| `tests/trade_stream.rs` | Live DEX swap streaming via Yellowstone gRPC |
| `tests/creation_stream.rs` | Live token + pool creation streaming |
| `tests/pool_financials.rs` | Live pool update streaming |
| `tests/validate_prices.rs` | Price validation across DEXs |

## Runtime / Tooling

- **Rust edition:** 2024 (requires rustc 1.85+)
- **Workspace resolver:** 3
- **7 workspace dependencies:** `serde`, `solana-sdk`, `solana-pubkey`, `solana-system-interface`, `spl-associated-token-account`, `spl-token`, `borsh`
- **Aggregator dependencies:** `tokio`, `futures`, `solana-rpc-client`, `indicatif`, `rustyline`, `sysinfo`, `bincode`
- **Engine dependencies:** `axum`, `dashmap`, `tower-http`, `yellowstone-grpc-client`, `yellowstone-grpc-proto`, `rustls`

---
> Source: [anshuman008/solana-dex-aggregator](https://github.com/anshuman008/solana-dex-aggregator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
