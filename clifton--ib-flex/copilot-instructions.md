## ib-flex

> `ib-flex` is a pure-Rust parser for Interactive Brokers **FLEX** (Flex Web Query)

# CLAUDE.md

`ib-flex` is a pure-Rust parser for Interactive Brokers **FLEX** (Flex Web Query)
XML statements — both **Activity FLEX** and **Trade Confirmation FLEX**. It is a
standalone, MIT-licensed crate published on crates.io. Keep it dependency-light
(`quick-xml`, `serde`, `rust_decimal`, `chrono`, `thiserror`; `reqwest`/`tokio`
only behind the `api-client` feature) and free of any coupling to other repos.

## Before every commit (required)

```bash
cargo fmt && cargo clippy --all-targets -- -D warnings
```

Both must be clean (zero warnings) before committing. Then run `cargo test`.

## Commands

```bash
cargo build                         # library (add --features api-client for the HTTP client)
cargo test                          # unit + integration + doctests
cargo test --doc                    # doctests only
cargo bench                         # benchmarks
# Optional smoke test against a real (private) statement; skipped if the env var is unset:
IB_FLEX_REAL_XML=path/to/statement.xml cargo test --test real_data_smoke -- --nocapture
```

Never commit real account XML — put it under `tmp/` (gitignored).

## Layout

- `src/lib.rs` — public API: `parse_activity_flex`, `parse_activity_flex_all`,
  `parse_trade_confirmation`, `detect_statement_type` (+ re-exports). `api::FlexApiClient`
  lives behind the `api-client` feature.
- `src/types/`
  - `common.rs` — shared enums (`AssetCategory`, `BuySell`, `OpenClose`, `OrderType`, `TradeType`, …).
  - `activity.rs` — `Trade`, `Position`, `CashTransaction`, `CorporateAction`, `SecurityInfo`,
    `ConversionRate`, the top-level `ActivityFlexStatement`, and the per-section wrapper structs.
  - `extended.rs` — the remaining element structs (transfers, accruals, performance summaries,
    SLB, fees, summaries, …).
  - `trade_confirmation.rs` — `TradeConfirmationStatement`.
- `src/parsers/`
  - `activity.rs`, `trade_confirmation.rs` — parse entry points (serde + quick-xml).
  - `xml_utils.rs` — the custom optional deserializers (see below).
- `tests/` — integration tests + XML fixtures. `wild_fixtures.rs` covers real-world-shaped
  edge cases; `real_data_smoke.rs` is the env-gated real-data test.

## Conventions and gotchas

- **`ibflex` is the ground truth for attribute names.** The Python library
  [csingley/ibflex](https://github.com/csingley/ibflex) has been validated against real IB
  output for years; its dataclass field names ARE the real camelCase XML attribute names.
  When adding or renaming a field, verify the attribute name there — synthetic fixtures have
  historically used wrong names (e.g. `price` instead of the real `tradePrice`).
- **Every attribute-derived field is `Option<T>`.** Real statements may omit any attribute, and
  a missing one must never fail the whole parse. Use the helpers in `xml_utils.rs`:
  - `Option<Decimal>` → `deserialize_optional_decimal` (strips thousands separators; `-`/`--`/`N/A`/`""` → `None`)
  - `Option<NaiveDate>` → `deserialize_optional_date` (accepts `yyyy-MM-dd` / `yyyyMMdd`, the `MULTI` sentinel, and a trailing time component)
  - `Option<String>` → `deserialize_optional_string` (`""` and sentinels → `None`)
  - `Option<bool>` → `deserialize_optional_bool` (`Y`/`N`/`Yes`/`No`)
  - enums → `#[serde(rename = "@x", default)]` (each enum has an `#[serde(other)] Unknown` catch-all)
- **Datetimes are kept as `Option<String>`** (IB format `yyyyMMdd;HHmmss`), not parsed to a chrono type.
- IB only emits dates as ISO-8601 (`yyyy-MM-dd`) or compact (`yyyyMMdd`); European `dd/MM/yyyy` is unsupported.
- The `<Trades>` container interleaves `<Trade>`, `<Lot>`, `<Order>`, `<SymbolSummary>`, and
  `<AssetSummary>` rows (all trade-shaped); they are captured into
  `TradesWrapper::{items, lots, orders, symbol_summaries, asset_summaries}`.
- `CorporateActionType` and `TransactionCode` enums exist but are currently **unwired** — the raw
  code string is captured instead. Wiring or removing them is an open follow-up.

## Release

1. Bump `version` in `Cargo.toml`; move CHANGELOG `[Unreleased]` to `[x.y.z]` with the date.
2. Commit `chore: Release vX.Y.Z`, tag `git tag -a vX.Y.Z -m "Release vX.Y.Z"`, push the commit + tag.
3. Pushing a `v*.*.*` tag triggers `.github/workflows/release.yml`, which verifies the tag matches
   `Cargo.toml` and runs `cargo publish`.
4. Create the matching GitHub Release for the tag.

---
> Source: [clifton/ib-flex](https://github.com/clifton/ib-flex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
