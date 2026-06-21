## ibcore

> - Rust (edition 2024)

# ibcore — Rust IB Gateway Integration

## Tech Stack
- Rust (edition 2024)
- `ibapi` crate v3.0 for Interactive Brokers Gateway
- Tokio async runtime

## Commands
- Compile: `cargo check`
- Full build: `cargo build`
- Unit tests: `cargo test --lib`
- Clippy lint: `cargo clippy -- -D warnings`
- Build with Python feature: `cargo build --features python`
- Python build: `cd python && maturin develop --release`

## Testing

### Unit Tests
```bash
cargo test --lib
```
131 tests covering: client, snapshots, diagnostics, errors, orders, streaming, historical, chain, contract, exchange.

### Python Tests
```bash
cd python && pytest tests/
```

## Conventions
- TDD: write failing test → minimum code → refactor
- All IB calls async via Tokio
- PyO3 bindings behind `#[cfg(feature = "python")]` gate
- No hardcoded secrets

---
> Source: [siongsheng/ibcore](https://github.com/siongsheng/ibcore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-21 -->
