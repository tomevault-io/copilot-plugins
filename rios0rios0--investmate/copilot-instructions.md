## investmate

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

InvestMate is a Go CLI application that fetches ETF financial data (dividend cash amounts, average closing prices) from external sources, calculates dividend yields per year, and renders results as a colour-coded ASCII table in the terminal.

## Build, Test, and Lint

```bash
# Build
make build                # Binary to bin/investmate
make run                  # Run directly (fetches live NASDAQ data)

# Test
go test ./...             # All tests (some call live NASDAQ APIs, requires network)
go test -race ./...       # With race detector
go test -run TestETFTestSuite ./internal/domain/entities/...  # Single suite
go test -run TestMain_ProcessETF ./cmd/...                    # Single test

# Quality gates (always use Makefile targets, never call tools directly)
make lint                 # Linting via pipelines repo
make sast                 # Full SAST suite (CodeQL, Semgrep, Trivy, Hadolint, Gitleaks)
```

## Architecture

Clean Architecture with Hexagonal (Ports & Adapters) pattern:

- **`cmd/main.go`** — Entry point. Wires repositories to entities, drives table rendering. Manual DI (no framework).
- **`internal/domain/`** — Contracts layer. Contains entity business logic and repository interfaces (ports).
- **`internal/infrastructure/`** — Implementations layer. Contains repository adapters for external data sources.

### Data Flow

`main()` → instantiates repository adapters → calls repository interfaces → populates ETF entity → entity calculates yields → renders ASCII table.

### Repository Interfaces (Ports)

- `DividendsRepository.ListDividendsByETF(etf string) (map[string]float64, error)`
- `PricesRepository.ListClosingPricesByETF(etf string) (map[string]float64, error)`

### Adapters

- **`nasdaq/`** — Active. REST API adapters for dividends and closing prices.
- **`statusinvest/`** and **`historyorg/`** — Currently unused web-crawler adapters.

### Key Dependencies

| Package | Purpose |
|---|---|
| `gocolly/colly` | HTML scraping |
| `olekukonko/tablewriter` | ASCII table rendering with ANSI colours |
| `sirupsen/logrus` | Structured logging |
| `stretchr/testify` | Test assertions and suites |

## Testing Conventions

- Tests use **testify** (`assert` for unit tests, `suite.Suite` for entity tests).
- BDD structure with `// given`, `// when`, `// then` comments.
- Sub-test names follow `"should ... when ..."` pattern.
- Unit tests use `t.Parallel()` at both parent and sub-test levels.
- Entity tests use `suite.Run()` (not parallel due to shared setup).
- Some tests call live NASDAQ APIs — expect network access requirement.

## Configuration

Compile-time constants in `cmd/main.go`:
- `YearsToFetch` (default: 5) — years of historical data
- `targetYieldPercentage` (default: 9) — minimum yield for green colouring
- `etfNames` slice in `main()` — list of ETF tickers to process

## Troubleshooting

- **Empty API data**: NASDAQ blocks requests without a browser-like `User-Agent` header. The repository sets one explicitly.
- **Build errors after `go mod tidy`**: Requires Go >= 1.26.

## CI/CD

Defined in `.github/workflows/default.yaml`, delegates to `rios0rios0/pipelines/.github/workflows/go-binary.yaml@main`. Triggers on push to main, tag push, PR to main, and manual dispatch.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rios0rios0) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
