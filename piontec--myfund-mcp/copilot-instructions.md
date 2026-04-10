## myfund-mcp

> AI agent guidance for this repository. Read this before making changes.

# AGENTS.md — myfund-mcp

AI agent guidance for this repository. Read this before making changes.

---

## What This Project Does

`myfund-mcp` is a **Rust MCP (Model Context Protocol) server** that exposes investment portfolio data from [myfund.pl](https://myfund.pl) as MCP tools. It is intended to be used with Claude Desktop, GitHub Copilot, or any MCP-compatible AI assistant.

---

## Project Structure

```
myfund-mcp/
├── AGENTS.md           ← you are here
├── Cargo.toml          ← dependencies (rmcp, tokio, reqwest, serde, anyhow, tracing, wiremock dev)
├── README.md           ← user-facing setup and config instructions
├── tests/
│   └── fixtures/
│       ├── portfolio_ok.json     ← realistic success response (3 tickers, all time-series)
│       └── portfolio_error.json  ← API error response (status.code = "1")
└── src/
    ├── main.rs         ← entry point: reads env vars, sets up tracing, starts stdio transport
    ├── server.rs       ← MCP ServerHandler with #[tool] macro implementations + unit tests
    ├── myfund.rs       ← HTTP client for the myfund.pl API + unit tests
    └── models.rs       ← serde data models for myfund.pl JSON responses + unit tests
```

---

## Architecture

```
stdin/stdout (MCP stdio)
       │
       ▼
  main.rs  ──── reads MYFUND_API_KEY, MYFUND_PORTFOLIOS env vars
       │         starts rmcp stdio transport
       ▼
  server.rs  ── ServerHandler impl
       │         three #[tool] methods (see below)
       ▼
  myfund.rs  ── MyfundClient
       │         single async fn: fetch_portfolio(name) → PortfolioResponse
       ▼
  myfund.pl API (HTTPS, API key auth via query param)
```

Tracing/logging goes to **stderr only** — stdout is reserved for the MCP stdio channel.

---

## MCP Tools

| Tool | Input | Returns |
|------|-------|---------|
| `list_portfolios` | — | Configured portfolio names (from `MYFUND_PORTFOLIOS` env var) |
| `get_portfolio_summary` | `name: String` | Portfolio totals + all ticker positions + asset structure + today's daily change. **No time-series** (response size constraint). |
| `get_portfolio_timeseries` | `name: String`, `series: String`, `from?: String (YYYY-MM-DD)`, `to?: String (YYYY-MM-DD)` | One named time-series, optionally filtered by date range |

Valid `series` values for `get_portfolio_timeseries`:
- `wartoscWCzasie` — daily portfolio value
- `zyskWCzasie` — daily profit
- `wkladWCzasie` — daily own contribution
- `benchWCzasie` — benchmark return
- `stopaZwrotuWCzasie` — portfolio rate of return

---

## myfund.pl API

- **Endpoint:** `GET https://myfund.pl/API/v1/getPortfel.php?portfel={name}&apiKey={key}&format=json`
- **Auth:** API key passed as query param (no sessions, no cookies)
- **Response size:** ~660 KB (includes full daily time-series since portfolio inception)
- **Server-side cache:** 5 minutes (repeated requests within 5 min return the same data)
- **Error response:** `{"status":{"code":"1","text":"<message>"}}`
- **Portfolio names are case-sensitive**

### Response JSON Shape

```
status              { code: "0"|"1", text: string }
portfel             { benchName, tickersCount, nazwa, waluta, wartosc, zysk, zyskDzienny,
                      zmianaDzienna, close, liczbaJednostek, udzial, zmiana,
                      zmianaW, zmiana2W, zmianaM, zmiana3M, zmiana6M, zmianaR,
                      zmiana3R, zmiana5R, zmianaMdD, zmianaRdD, ... }
                    NOTE: mixed types — some fields are f64, some are String
tickers             HashMap<numeric_string_id, Ticker>
                    Ticker: { tickerClear, nazwa, data, close, zmianaDzienna,
                              liczbaJednostek, typ, typOrg, wartosc, udzial, zmiana,
                              cenaZakupu, zysk, kontoInvName, sektor, ryzyko,
                              portfelOrg, dataInvStart, okresInwestycji }
                    NOTE: all Ticker fields are strings
struktura           HashMap<asset_type_name, value_string>
strukturaWalory     HashMap<ticker_name, share_string>
strukturaKolor, strukturaWaloryKolor  (color maps, can be ignored)
wartoscWCzasie      HashMap<"YYYY-MM-DD", value_string>
zyskWCzasie         HashMap<"YYYY-MM-DD", value_string>
wkladWCzasie        HashMap<"YYYY-MM-DD", value_string>
benchWCzasie        HashMap<"YYYY-MM-DD", value_string>
stopaZwrotuWCzasie  HashMap<"YYYY-MM-DD", value_string>
zmianaDzienna       HashMap<"YYYY-MM-DD", value_string>  (usually just today)
```

---

## Test Fixtures

Located in `tests/fixtures/`:

| File | Description |
|------|-------------|
| `portfolio_ok.json` | Realistic success response — 3 tickers (NASDAQ stock, WSE stock, international ETF), all 5 time-series with 8 data points each, struktura, zmianaDzienna. Field types mirror the real API (mixed numeric/string in `portfel`, all-string in `tickers`). |
| `portfolio_error.json` | API error response with `status.code = "1"` and Polish error message |

Load in tests with: `include_str!("../tests/fixtures/portfolio_ok.json")`

### Unit Test Summary (30 tests across 3 modules)

- **`models.rs`** (12 tests): deserialization of both fixtures, field value assertions, ticker count/fields, struktura keys, time-series entries, `is_error()` helper
- **`myfund.rs`** (6 tests): HTTP mock tests with `wiremock` — success path, API error propagation, HTTP 500, query param correctness
- **`server.rs`** (12 tests): tool output content (portfel/tickers present, time-series absent in summary), date range filtering (from, range, empty), invalid series name, error propagation, list_portfolios with/without env var

---

## Key Dependencies

| Crate | Version | Purpose |
|-------|---------|---------|
| `rmcp` | 0.16 | Official Rust MCP SDK; features: `server`, `macros`, `schemars`, `transport-io` |
| `tokio` | 1 | Async runtime (`features = ["full"]`) |
| `reqwest` | 0.12 | HTTP client for myfund.pl API (`features = ["json"]`) |
| `serde` / `serde_json` | 1 | JSON (de)serialization |
| `anyhow` | 1 | Error handling |
| `tracing` + `tracing-subscriber` | 0.1 / 0.3 | Structured logging to stderr |
| `wiremock` *(dev)* | 0.6 | HTTP mock server for `myfund.rs` unit tests |

---

## Configuration

| Env Var | Required | Description |
|---------|----------|-------------|
| `MYFUND_API_KEY` | ✅ | API key from myfund.pl: menu → Account → Account Settings → API Key |
| `MYFUND_PORTFOLIOS` | Optional | Comma-separated portfolio names, e.g. `main,crypto,etf` |

The server exits with a clear error message if `MYFUND_API_KEY` is not set.

---

## Coding Conventions

- Use `anyhow::Result` for error propagation throughout.
- Tool methods on `MyfundServer` use the `#[tool]` proc-macro from `rmcp-macros`.
- `serde_json::Value` is used for the `portfel` summary fields due to mixed types in the API response.
- All `Ticker` struct fields are `Option<String>` to handle missing fields gracefully.
- Time-series types are `HashMap<String, String>` (date → value).
- Do not add client-side caching — the myfund.pl API already caches for 5 minutes.
- Do not write to stdout from application code — it breaks the MCP stdio channel.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/piontec)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/piontec)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
