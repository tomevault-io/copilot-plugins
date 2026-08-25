## turnstile

> - Always run e2e tests before declaring proxy changes complete. Everything you need is available locally.

# Turnstile

## Testing
- Always run e2e tests before declaring proxy changes complete. Everything you need is available locally.
- Quick start: `make e2e-up` starts sglang + turnstile, then iterate with `cargo test --manifest-path tests/e2e/Cargo.toml --test turnstile -- --nocapture`.
- Full run: `make e2e` (starts sglang, builds+starts turnstile, runs all tests, cleans up).
- Port 8080 may already be in use locally — use `TURNSTILE_LISTEN_ADDR=0.0.0.0:8081` and `TURNSTILE_URL=http://localhost:8081` to override.
- Leave sglang running between iterations (`make e2e-up` / `make e2e-down`) — model load is the slow part.

---
> Source: [amazon-agi-labs/turnstile](https://github.com/amazon-agi-labs/turnstile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
