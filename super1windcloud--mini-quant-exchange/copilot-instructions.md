## mini-quant-exchange

> This repository is a Rust workspace for a teaching-oriented mini quant exchange. Core domain crates live at the workspace root: `orderbook`, `matching-engine`, `kline-generator`, `market-data`, `risk-engine`, `strategy-engine`, `backtest-engine`, and `web-ui`. Shared entrypoints are in `src/`: `main.rs` runs the local demo, `lib.rs` builds dashboard HTML, and `src/bin/` contains focused binaries such as `binance_market`, `binance_strategy`, and `web_demo`. Generated browser output is written to `web-ui/demo-dashboard.html`. Ignore `target/` build artifacts and avoid committing IDE-only changes from `.idea/`.

# Repository Guidelines

## Project Structure & Module Organization
This repository is a Rust workspace for a teaching-oriented mini quant exchange. Core domain crates live at the workspace root: `orderbook`, `matching-engine`, `kline-generator`, `market-data`, `risk-engine`, `strategy-engine`, `backtest-engine`, and `web-ui`. Shared entrypoints are in `src/`: `main.rs` runs the local demo, `lib.rs` builds dashboard HTML, and `src/bin/` contains focused binaries such as `binance_market`, `binance_strategy`, and `web_demo`. Generated browser output is written to `web-ui/demo-dashboard.html`. Ignore `target/` build artifacts and avoid committing IDE-only changes from `.idea/`.

## Build, Test, and Development Commands
- `cargo check --workspace`: fast workspace-wide compile validation.
- `cargo test --workspace`: runs all unit and doc tests.
- `cargo run`: runs the local exchange and backtest demo.
- `just check` / `just test`: shortcuts for the main validation commands.
- `just binance-market BTCUSDT 10`: fetches Binance spot market data.
- `just binance-strategy BTCUSDT 30`: evaluates live strategy signals from Binance data.
- `just web-demo`: starts the Rust static demo server at `127.0.0.1:3000`.

## Coding Style & Naming Conventions
Use standard Rust formatting: 4-space indentation, `snake_case` for functions/modules, `CamelCase` for types, and `SCREAMING_SNAKE_CASE` for constants. Keep crates narrowly scoped by responsibility. Prefer explicit domain names such as `BacktestResult`, `RiskLimits`, or `BinanceSpotMarketClient` over generic helpers. Run `cargo fmt` before submitting changes; keep comments brief and only where logic is not obvious.

## Testing Guidelines
Place unit tests beside implementation in each crate’s `src/lib.rs` using `#[cfg(test)]`. Name tests by behavior, for example `matches_crossing_orders` or `rejects_position_breach`. New trading logic should include at least one deterministic test covering success and rejection paths. Run `cargo test --workspace` before opening a PR.

## Commit & Pull Request Guidelines
Recent history uses short, imperative English commit messages such as `Add Binance strategy and web demo`. Follow that pattern: start with a verb, describe the user-visible change, and keep the subject concise. PRs should include a summary, affected crates, validation steps run, and screenshots or sample URLs when changing `web-ui` or `web_demo`.

## Security & Configuration Tips
Binance public market reads do not require secrets in the current implementation. If you add signed endpoints later, load credentials from environment variables and never hardcode keys. Treat all external market timestamps and decimal scales carefully; this repo uses integer scaling for prices and quantities.

---
> Source: [Super1Windcloud/mini-quant-exchange](https://github.com/Super1Windcloud/mini-quant-exchange) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
