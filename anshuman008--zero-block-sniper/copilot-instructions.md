## zero-block-sniper

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run Commands

```bash
cargo build                          # Build the library + bot
cargo run --bin sniper-bot           # Run sniper bot
cargo run --example basic            # Run basic example (all events)
cargo run --example trades_only      # Run trades-only example
cargo run --example new_tokens       # Run new tokens example
cargo run --example sniper           # Run sniper example
cargo run --example pumpfun          # Run pumpfun example
cargo check                          # Type-check without building
cargo clippy                         # Lint
```

No test suite exists. Examples and bot require a Yellowstone gRPC endpoint and optional auth token (loaded via `.env`).

## Architecture

This is a Rust library (`rlib` + `cdylib`) for real-time PumpFun event streaming on Solana via Yellowstone gRPC (Geyser plugin), plus a production sniper bot binary.

### Data Flow

```
Yellowstone gRPC → PumpFunClient (client.rs)
  → process_transaction / process_account (processor.rs)
    → parse_instruction / parse_inner_instruction / parse_account (parser.rs)
      → merge_events (merges instruction data with CPI log data)
        → callback(PumpFunEvent)
```

### Core Library Modules

- **`streaming/client.rs`** — `PumpFunClient`: connects to gRPC, manages subscription lifecycle (start/stop), dispatches raw updates to processor. Uses `tokio::select!` for stop signal + stream processing.
- **`streaming/processor.rs`** — Extracts PumpFun instructions from gRPC transaction updates. Resolves account keys (including loaded addresses), matches outer instructions to inner CPI logs, and calls `merge_events` to combine both data sources into a complete event.
- **`parser.rs`** — Stateless parsing: routes discriminators (8-byte for instructions, 16-byte for CPI logs, 8-byte for accounts) to type-specific parsers. `merge_events()` enriches instruction-level events with CPI log data (reserves, timestamps, fees). CreateTokenV2 uses accounts[5]=user, accounts[7]=token_program per IDL.
- **`events.rs`** — Event structs (`CreateTokenEvent`, `CreateTokenV2Event`, `TradeEvent`, `MigrateEvent`, `BondingCurveAccountEvent`, `GlobalAccountEvent`). Uses `#[borsh(skip)]` to separate instruction-level fields from CPI log fields on the same struct.
- **`types.rs`** — `EventMetadata`, `BondingCurve` (with price/mcap/progress calculations), `Global` config, `AccountData` wrapper. Constants: `BONDING_CURVE_SIZE`, `GLOBAL_SIZE`.
- **`sniper.rs`** — Zero-RPC buy/sell instruction builder. Derives all accounts (ATA, PDA vaults) from stream event data. `build_buy_ix_from_event` takes `bc_creator` (bonding curve creator) for correct `creator_vault` PDA derivation. Includes `calculate_tokens_out` for AMM math.
- **`streaming/config.rs`** — `StreamConfig`, `TransactionFilter`, `AccountFilter` with PumpFun-only defaults.

### Sniper Bot Modules (`src/bin/sniper_bot/`)

- **`main.rs`** — Orchestrator: wires event sources (shred streams + gRPC) → dedup → rate limit → build tx → submit via Jito.
- **`config.rs`** — `BotConfig::from_env()`. All 8 Jito regions enabled by default. Configurable via `JITO_REGIONS`, `RATE_LIMIT_MS`, etc.
- **`jito_bundle.rs`** — Builds single-tx with embedded tip (no separate tip tx). Submits to all Jito regions in parallel via both `sendBundle` and `sendTransaction`.
- **`blockhash.rs`** — Background task refreshing blockhash at configurable interval.
- **`dedup.rs`** — `DedupSet`: TTL-based mint deduplication using `DashMap`.
- **`shred_stream.rs`** — Connects to Jito ShredStream and/or Helius LaserStream for zero-block latency.
- **`shred_parser.rs`** — Extracts PumpFun CreateTokenV2 events from raw shred entry bytes.

### Key Design Patterns

- **Two-phase parsing**: Outer instructions provide account addresses; inner CPI logs provide computed values (reserves, fees, timestamps). `merge_events()` combines both into one `PumpFunEvent`.
- **Discriminator-based routing**: 8-byte discriminators for instructions/accounts, 16-byte for CPI log events. All defined in `events::discriminators`.
- **`PumpFunEvent` enum** is the unified public type — all variants carry `EventMetadata` with slot, signature, and event type.
- **Zero-RPC sniping**: All PDAs derived from event data. `creator_vault` uses `event.user` (deployer = bonding curve creator). Only RPC call is background blockhash refresh.
- **Embedded Jito tip**: Single tx `[CU_limit, create_ata, buy_ix, tip_transfer]` — no separate tip transaction, eliminates uncle bandit risk.
- **Multi-region fan-out**: Parallel submission to 8 Jito block engines via both `sendBundle` and `sendTransaction` endpoints.

### Program ID

PumpFun: `6EF8rrecthR5Dkzon8Nwu78hRvfCKubJ14M5uBEwF6P`

### Key Dependencies

- `yellowstone-grpc-client` / `yellowstone-grpc-proto` — gRPC streaming
- `solana-sdk` / `solana-client` — Solana primitives
- `reqwest` — HTTP client for Jito bundle API
- `borsh` — On-chain data deserialization
- `dashmap` — Lock-free concurrent dedup map

---
> Source: [anshuman008/zero-block-sniper](https://github.com/anshuman008/zero-block-sniper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
