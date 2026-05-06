## ai-oracle-skills

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repo contains **Claude Code skills** for the APRO AI Oracle Ticker API — a cryptocurrency price, category, and OHLCV data oracle with cryptographic signatures for on-chain verification.

There are two skills in `skills/`:

- **ai-oracle-integration** (`skills/ai-oracle-integration/`) — Code generation skill that produces production-ready client libraries for the Ticker API in any language. Generated code goes to `examples/<language>/ticker/` with a mandatory directory structure (`src/types/`, `src/client/`, `src/utils/`, `examples/`). The OpenAPI spec at `skills/ai-oracle-integration/references/ai-oracle-api.yaml` is the authoritative source for all types and endpoints.

- **ticker-query-skill** (`skills/ticker-query-skill/`) — Live query skill that calls the Ticker API directly via curl and returns formatted results. No code generation — just HTTP requests.

## API Essentials

- **Base URL**: `https://api-ai-oracle.apro.com`
- **Auth**: `X-API-KEY` and `X-API-SECRET` headers on every request
- **Critical naming**: API uses full cryptocurrency names (`name=Bitcoin`), NOT symbols (`BTC`). Quote currency param is `quotation`, not `currency`. OHLCV intervals are `"hourly"`/`"daily"`, not `"1h"`/`"1d"`. OHLCV query timestamps are in **milliseconds**, response timestamps in **seconds**.
- **6 endpoints**: currencies/list, currency/support, currency/price, category/list, category/coins, currency/ohlcv

## Testing

Run the API test script against all 6 endpoints:
```bash
# Requires APRO_API_KEY and APRO_API_SECRET env vars (or uses defaults in script)
chmod +x test_ticker_api.sh && ./test_ticker_api.sh
```

## Code Generation Verification

When the ai-oracle-integration skill generates code, it must pass compilation:

| Language | Command |
|----------|---------|
| TypeScript | `cd examples/<lang>/ticker && npm install && npx tsc --noEmit` |
| Python | `python3 -m py_compile src/types/index.py && ...` (each file) |
| Go | `cd examples/go/ticker && go mod tidy && go build ./... && go vet ./...` |
| Rust | `cd examples/rust/ticker && cargo check` |

---
> Source: [APRO-com/ai-oracle-skills](https://github.com/APRO-com/ai-oracle-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
