## fmp-rs

> `fmp-agent` is an unofficial CLI for the Financial Modeling Prep stable API, optimized for predictable JSON output in shell pipelines and LLM tool-calling. It is not affiliated with, endorsed by, or sponsored by Financial Modeling Prep.

# fmp-agent CLI reference

`fmp-agent` is an unofficial CLI for the Financial Modeling Prep stable API, optimized for predictable JSON output in shell pipelines and LLM tool-calling. It is not affiliated with, endorsed by, or sponsored by Financial Modeling Prep.

This document is the long-form reference. For programmatic discovery, prefer the `schema` subcommand below.

## 1. Self-discovery (start here)

LLMs and tool runners should discover the command surface from the binary itself instead of parsing this document:

```bash
fmp-agent doctor             # JSON readiness: version, base URL validity, API key presence
fmp-agent commands           # list all leaf command paths, one per line
fmp-agent completions <shell> # generate bash/zsh/fish/powershell completions script
fmp-agent schema             # versioned JSON: { schema_version, binary, version, commands[] }
fmp-agent help environment   # environment variables, .env, and config precedence
fmp-agent help exit-codes    # exit codes and structured stderr errors
fmp-agent help schema        # schema and tool-calling guidance
fmp-agent help examples      # representative command examples
fmp-agent --help             # human-readable top-level help
fmp-agent <GROUP> --help     # human-readable per-group help
fmp-agent <GROUP> <CMD> --help  # human-readable per-command help, includes Examples
```

`fmp-agent doctor`, `fmp-agent commands`, `fmp-agent completions`, `fmp-agent schema`, and `fmp-agent help <topic>`:

- Do **not** require `FMP_API_KEY` and make **no** network requests.
- `doctor` emits JSON with `ok`, `version`, sanitized `base_url`, `api_key.configured`, and `live_connectivity.checked: false` so callers can verify local readiness without consuming quota.
- `commands` prints one leaf path per line, sorted alphabetically (e.g. `analyst grades`, `company profile`).
- `completions` generates a shell completions script (bash/zsh/fish/powershell) on stdout.
- `schema` emits `schema_version: 3` today; the shape is experimental and may change between releases.
- `help <topic>` prints operational guidance from the installed binary. Topics are `environment`, `exit-codes`, `schema`, and `examples`.
- Each schema command entry has `name`, `path`, `aliases`, `preferred_path`, `api_key_required`, `about`, `long_about`, and `args`.
- Each arg has `name`, `kind`, `required`, `default`, `value_name`, `help`, `long` (exact `--flag` spelling or null for positional), `short` (single-char flag or null), `parser` (type hint: `string`, `integer`, `bool`, `enum`, or `count`), `possible_values` (array of `{"name": "...", "help": "..."}` for enum args, null otherwise), and `multi_value` (whether the arg accepts repeat values).
- Use these as the source of truth for available commands and arguments; treat the catalog in section 6 as a curated index.

Top-level aliases (no API key required for discovery):

```bash
fmp-agent quote AAPL         # alias for market quote
fmp-agent historical AAPL    # alias for market historical
fmp-agent profile AAPL       # alias for company profile
fmp-agent earnings           # alias for calendar earnings
```

## 2. Install

```bash
cargo install rusty-fmp --locked
```

GitHub releases also publish cargo-dist archives and shell or PowerShell installers. The Cargo package is `rusty-fmp`; the installed binary is `fmp-agent`. From a repo checkout, substitute `cargo run -- <GROUP> <CMD>` for `fmp-agent <GROUP> <CMD>`.

## 3. Invocation contract

```bash
fmp-agent [OPTIONS] <GROUP> <CMD> [ARGS]
```

Stable behavior callers can rely on:

- **Success**: the raw FMP JSON payload on **one line** to stdout, exit code 0.
- **Runtime error** (config, network, API, parse, strict empty result): JSON envelope on **stderr**, non-zero exit:
  `{"ok": false, "error": {"kind": "...", "message": "..."}}`
- **Rate limit**: HTTP 429 exits 5 with `error.kind` set to `rate_limited`. Retry later with backoff instead of treating it as a subscription, authentication, or generic API failure.
- **Parse error** (bad flags or missing required args): Clap's human-readable usage text on stderr, exit code 2. To distinguish programmatically, check exit code first; only parse stderr as JSON for codes 3-7.
- Empty JSON arrays are successful raw FMP responses by default because they can be valid for date ranges, news windows, or unknown symbols. For symbol lookups where an empty result should stop automation, pass `--strict-empty`; it exits 7 with `empty_result` and suggests `fmp-agent search <SYMBOL>`.
- Help and version output are human-readable text, not JSON.
- The CLI deliberately offers no output formatting, filtering, or pagination flags. Pipe through `jq` for selection.
- Running `fmp-agent` with no command prints help and exits.

### Exit codes

| Code | Meaning                                                        |
| ---- | -------------------------------------------------------------- |
| 0    | Success                                                        |
| 2    | Usage error (Clap parse failure)                               |
| 3    | Configuration error (missing API key or invalid base URL)      |
| 4    | Network error (HTTP request failed)                            |
| 5    | API error (server returned non-2xx, including rate limits)     |
| 6    | Parse error (JSON deserialization failed)                      |
| 7    | Empty symbol result in `--strict-empty` mode                   |

Run `fmp-agent help exit-codes` for the same exit-code guidance from the binary.

## 4. Configuration

Global options apply to every subcommand:

| Flag                  | Env var          | Default                                          | Notes                                                          |
| --------------------- | ---------------- | ------------------------------------------------ | -------------------------------------------------------------- |
| `--api-key <KEY>`     | `FMP_API_KEY`    | (none; required for API commands)                | Not required for metadata commands (`commands`, `completions`, `schema`, `help <topic>`). Prefer env or `.env`. |
| `--base-url <URL>`    | `FMP_BASE_URL`   | `https://financialmodelingprep.com/stable/`      | Override for proxies or tests.                                 |
| `--strict-empty`      |                  | `false`                                          | Fail empty symbol lookup responses and suggest `search`.        |
| `-v` / `-vv` / `-vvv` | `RUST_LOG`       | warnings only                                    | INFO / DEBUG / TRACE; logs go to stderr; API key is redacted.  |
| `-h`, `--help`        |                  |                                                  | Human-readable help.                                           |
| `-V`, `--version`     |                  |                                                  | Binary version.                                                |

A `.env` file in the working directory is loaded automatically before parsing.
Run `fmp-agent help environment` for the same configuration guidance from the binary.

## 5. Argument shapes

Commands fall into a small set of reusable argument shapes. Pattern-matching on shape is the fastest way to generalize across the catalog.

| Shape                  | Positional | Options                                                  | Typical use                            |
| ---------------------- | ---------- | -------------------------------------------------------- | -------------------------------------- |
| `Endpoint`             | (none)     | (none)                                                   | List endpoints (`market stock-list`).  |
| `Query`                | `<QUERY>`  | (none)                                                   | Free-text search.                      |
| `Symbol`               | `<SYMBOL>` | (none)                                                   | Reference data for one ticker.         |
| `SymbolLimit`          | `<SYMBOL>` | `--limit <N>`                                            | Recent rows for one ticker.            |
| `SymbolDateRange`      | `<SYMBOL>` | `--from <YYYY-MM-DD>` `--to <YYYY-MM-DD>`                | Time-series for one ticker.            |
| `DateRange`            | (none)     | `--from <YYYY-MM-DD>` `--to <YYYY-MM-DD>`                | Cross-market calendars and rates.      |
| `NameDateRange`        | `<NAME>`   | `--from` `--to`                                          | Indicator series by FMP indicator name.|
| `Annual`               | `<SYMBOL>` | `--limit <N>`                                            | Annual statements and metrics.         |
| `AnnualReportForm`     | `<SYMBOL>` | `--year <YEAR>` `[--period <PERIOD>]`                    | Single annual report form.             |
| `TechnicalSma`         | `<SYMBOL>` | `[--period-length <N>]` `[--timeframe <TF>]`             | Technical indicators.                  |
| `StockNews`            | `<SYMBOL>` | `--limit <N>`                                            | Per-symbol news.                       |
| `Paged`                | (none)     | `--page <N>` `--limit <N>`                               | Paginated feeds.                       |

All date arguments are inclusive `YYYY-MM-DD`. Symbols are FMP-style tickers (`AAPL`, `BTCUSD`, `EURUSD`).

### Defaults

These defaults are emitted by Clap (`[default: N]`) in `--help` and reflected in `schema` output. Listed once here for convenience.

| Argument                                              | Default         |
| ----------------------------------------------------- | --------------- |
| `Annual --limit`                                      | 5               |
| `SymbolLimit --limit` (`company historical-rating`)   | 5               |
| `StockNews --limit`                                   | 10              |
| `Paged --page`                                        | 0               |
| `Paged --limit`                                       | 10              |
| `AnnualReportForm --period`                           | `FY`            |
| `TechnicalSma --period-length`                        | 10              |
| `TechnicalSma --timeframe`                            | `1day`          |
| `sec filings --from` (computed at runtime)            | 90 days ago     |

## 6. Command catalog

Commands use the `<group> <verb>` form introduced in 0.4.0. See the README migration note for the 0.4.0 restructure.

### Discovery

| Command                 | Shape       |
| ----------------------- | ----------- |
| `doctor`                | `Endpoint` (no API key)|
| `help environment`      | `Endpoint` (no API key)|
| `help exit-codes`       | `Endpoint` (no API key)|
| `help schema`           | `Endpoint` (no API key)|
| `help examples`         | `Endpoint` (no API key)|
| `search <QUERY>`        | `Query`     |
| `schema`                | `Endpoint` (no API key)|

### Company reference

| Command                                        | Shape          |
| ---------------------------------------------- | -------------- |
| `company profile <SYMBOL>`                     | `Symbol`       |
| `company executives <SYMBOL>`                  | `Symbol`       |
| `company peers <SYMBOL>`                       | `Symbol`       |
| `company scores <SYMBOL>`                      | `Symbol`       |
| `company float <SYMBOL>`                       | `Symbol`       |
| `company rating <SYMBOL>`                      | `Symbol`       |
| `company historical-rating <SYMBOL>`           | `SymbolLimit`  |
| `company outlook <SYMBOL>`                     | Unavailable *  |

### Market data

| Command                                        | Shape              |
| ---------------------------------------------- | ------------------ |
| `market quote <SYMBOL>`                        | `Symbol`           |
| `market batch-quote <SYMBOLS...>`              | `Symbols`          |
| `market historical <SYMBOL>`                   | `SymbolDateRange`  |
| `market dividends <SYMBOL>`                    | `Symbol`           |
| `market splits <SYMBOL>`                       | `Symbol`           |
| `market price-change <SYMBOLS...>`             | `Symbols`          |
| `market stock-list`                            | `Endpoint`         |
| `market realtime-quote <SYMBOL>`               | `Symbol`           |
| `etf holdings <SYMBOL>`                        | `Symbol` *         |

\* `company outlook` is legacy-only in current FMP docs and returns `endpoint_unavailable` locally. `etf holdings` is intentionally exposed even though Starter accounts receive an API error; it exercises the structured error path (exit code 5).

### Crypto and forex

| Command                                        | Shape              |
| ---------------------------------------------- | ------------------ |
| `crypto list`                                  | `Endpoint`         |
| `crypto quote <SYMBOL>`                        | `Symbol`           |
| `crypto historical <SYMBOL>`                   | `SymbolDateRange`  |
| `forex quote <SYMBOL>`                         | `Symbol`           |
| `forex historical <SYMBOL>`                    | `SymbolDateRange`  |

### Fundamentals

| Command                                                          | Shape               |
| ---------------------------------------------------------------- | ------------------- |
| `fundamentals income-statement <SYMBOL>`                         | `Annual`            |
| `fundamentals income-statement-as-reported <SYMBOL>`             | `Annual`            |
| `fundamentals balance-sheet <SYMBOL>`                            | `Annual`            |
| `fundamentals cash-flow <SYMBOL>`                                | `Annual`            |
| `fundamentals ratios <SYMBOL>`                                   | `Annual`            |
| `fundamentals metrics <SYMBOL>`                                  | `Annual`            |
| `fundamentals income-statement-growth <SYMBOL>`                  | `Annual`            |
| `fundamentals balance-sheet-growth <SYMBOL>`                     | `Annual`            |
| `fundamentals cash-flow-growth <SYMBOL>`                         | `Annual`            |
| `fundamentals enterprise-values <SYMBOL>`                        | `Annual`            |
| `fundamentals analyst-estimates <SYMBOL>`                        | `Annual`            |
| `fundamentals report-dates <SYMBOL>`                             | `Symbol`            |
| `fundamentals annual-report <SYMBOL> --year <YEAR>`             | `AnnualReportForm`  |
| `fundamentals earnings <SYMBOL>`                                 | `SymbolLimit`       |
| `fundamentals statement-growth <SYMBOL>`                         | `Annual`            |

Income statement, balance sheet, and cash flow fundamentals accept `--period annual` (default) or `--period quarter` plus `--limit <N>` (default 5). Use quarterly periods for recent statement context, for example `fundamentals income-statement WOLF --period quarter --limit 4`. Other annual fundamentals commands keep the annual `--limit <N>` shape until quarterly access is confirmed in `docs/api-inventory.md`.

### Analyst

| Command                                          | Shape    |
| ------------------------------------------------ | -------- |
| `analyst price-target-consensus <SYMBOL>`        | `Symbol` |
| `analyst price-target-summary <SYMBOL>`          | `Symbol` |
| `analyst grades <SYMBOL>`                        | `Symbol` |
| `analyst price-target <SYMBOL>`                  | Unavailable * |
| `analyst upgrades-downgrades <SYMBOL>`           | Unavailable * |
| `analyst earnings-surprises <SYMBOL>`            | Unavailable * |

\* These analyst commands correspond to legacy-only FMP docs and return `endpoint_unavailable` locally. Use `analyst price-target-consensus`, `analyst price-target-summary`, and `analyst grades` for stable analyst data. If an older installed CLI returns `api_error` with `HTTP 404: []` for one of these commands, run `fmp-agent --version`, update the binary, or run `fmp-agent help troubleshooting` for the installed guidance.

### Calendars, rates, technicals, filings

| Command                                          | Shape              |
| ------------------------------------------------ | ------------------ |
| `calendar earnings`                              | `DateRange`        |
| `calendar market-hours`                          | `Endpoint`         |
| `macro treasury-rates`                           | `DateRange`        |
| `macro economic-indicators <NAME>`               | `NameDateRange`    |
| `technical sma <SYMBOL>`                         | `TechnicalSma`     |
| `sec filings <SYMBOL>`                           | `SymbolDateRange`  |
| `insider latest`                                 | `Paged`            |

`macro economic-indicators` takes an FMP indicator name as the positional, e.g. `GDP`, `CPI`.

### News

| Command                  | Shape       |
| ------------------------ | ----------- |
| `news stock <SYMBOL>`    | `StockNews` |
| `news general`           | `Paged`     |
| `news articles`          | `Paged`     |
| `news forex`             | `Paged`     |
| `news crypto`            | `Paged`     |

## 7. Examples

```bash
# Discovery
fmp-agent doctor | jq '{ok, base_url, api_key}'
fmp-agent schema | jq '.commands | map(.name)'
fmp-agent search Apple

# Reference data
FMP_API_KEY=your-key fmp-agent market quote AAPL
fmp-agent market batch-quote AAPL MSFT GOOGL
fmp-agent company profile AAPL
fmp-agent company historical-rating AAPL --limit 20

# Time series
fmp-agent market historical AAPL --from 2025-01-01 --to 2025-01-31
fmp-agent crypto historical BTCUSD --from 2025-01-01 --to 2025-01-03
fmp-agent forex historical EURUSD --from 2025-01-01 --to 2025-01-03
fmp-agent technical sma AAPL --period-length 20 --timeframe 1day

# Fundamentals
fmp-agent fundamentals income-statement AAPL --limit 5
fmp-agent fundamentals income-statement WOLF --period quarter --limit 4
fmp-agent fundamentals earnings WOLF --limit 4
fmp-agent fundamentals annual-report AAPL --year 2022

# Calendars and macros
fmp-agent calendar earnings --from 2026-01-01 --to 2026-01-31
fmp-agent macro treasury-rates --from 2025-01-01 --to 2025-01-31
fmp-agent macro economic-indicators GDP --from 2025-01-01 --to 2025-12-31

# Filings and news
fmp-agent sec filings AAPL --from 2024-01-01 --to 2024-03-01
fmp-agent news stock AAPL --limit 10
fmp-agent news general --page 0 --limit 10

# Verbose logging to stderr (does not affect stdout JSON)
fmp-agent -vv market quote AAPL 2> debug.log
```

## 8. Library use

Other Rust crates can depend on `rusty-fmp` as an HTTP client without the CLI. See the project `README.md` for the `default-features = false` recipe; the library re-exports `FmpClient`, `Endpoint`, `Error`, and `Result`. New endpoints are added by registering an `Endpoint` constant and dispatching through shape-based methods (`endpoint`, `query`, `by_symbol`, `by_symbol_limit`, `by_symbol_date_range`, `by_date_range`, `by_name_date_range`, `annual`, `annual_report_form`, `technical`, `news`, `paged`).

## 9. Development

```bash
make check          # fmt, clippy, tests, docs across both feature shapes
make coverage       # cargo llvm-cov; fails under 90% line coverage
make patch-coverage # diff-cover against PATCH_COVERAGE_BASE (default: main)
make audit          # cargo audit
make machete        # cargo machete (unused deps)
```

CI mirrors `make check` across Linux, macOS, and Windows, with an MSRV job pinned to Rust 1.96. Keep command help strings in `src/cli/help.rs` so `--help`, generated man pages, `schema` output, and this reference stay aligned.

---
> Source: [major/fmp-rs](https://github.com/major/fmp-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
