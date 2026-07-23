## khunquant

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

KhunQuant is an ultra-lightweight personal AI assistant written in Go. It targets minimal hardware ($10 devices) with <10MB RAM and <1 second boot time. It supports multiple LLM providers (Anthropic, OpenAI, Azure, Ollama, local models) and multiple chat platforms (Telegram, Discord, WeChat, Feishu, Matrix, IRC, etc.).

## Common Commands

```bash
# Development
make deps              # Download and verify dependencies
make build             # Build khunquant binary for current platform
make run ARGS="..."    # Build and run with arguments
make test              # Run all tests
make check             # Full pre-commit check (deps + fmt + vet + test)

# Code quality
make fmt               # Format code
make lint              # Run golangci-lint
make vet               # Run go vet
make fix               # Auto-fix linting issues

# Single test
go test ./pkg/agent/... -run TestName

# Cross-platform builds
make build-all         # All platforms (linux/amd64, arm, arm64, mips, riscv64, darwin, windows)
make build-pi-zero     # Raspberry Pi Zero 2 W

# Web launcher
cd web && make dev     # Start frontend + backend dev servers
cd web && make build   # Build frontend and embed into Go binary

# Docker
make docker-build      # Alpine-based minimal image
make docker-build-full # Full image with Node.js 24 (MCP support)
make docker-run        # Run gateway in Docker
```

## Architecture

### Entry Points
- `cmd/khunquant/` — Main agent binary using Cobra CLI. Subcommands: `onboard`, `agent`, `auth`, `gateway`, `status`, `cron`, `migrate`, `skills`, `model`, `version`
- `cmd/khunquant-launcher-tui/` — Terminal UI launcher

### Core Packages (`pkg/`)

**Agent Core:**
- `pkg/agent/` — Core AI agent loop (`loop.go`), context management (`context.go`), instance management (`instance.go`), memory (`memory.go`)
- `pkg/providers/` — LLM provider abstraction with ~40 subdirectories. Each provider implements a common interface. Supports Anthropic, OpenAI, Azure, Ollama, and local models.
- `pkg/channels/` — Chat platform adapters (Discord, Telegram, WeChat, Feishu, Matrix, IRC, Line, OneBot, etc.)
- `pkg/tools/` — Agent tool implementations (filesystem, shell, editing, cron, search, MCP, I2C, etc.)
- `pkg/skills/` — Extensible skills framework
- `pkg/commands/` — Built-in agent commands (help, check, list, switch, etc.)

**Infrastructure:**
- `pkg/config/` — Centralized configuration with migration support (`config.go`, `migration.go`, `defaults.go`)
- `pkg/routing/` — Message routing between channels
- `pkg/session/` — User session handling
- `pkg/bus/` — Event bus for inter-component communication
- `pkg/mcp/` — Model Context Protocol support
- `pkg/memory/` — Agent memory and state persistence (SQLite via `modernc.org/sqlite`)
- `pkg/logger/` — Structured logging (zerolog-based)
- `pkg/cron/` — Scheduled task execution

**Web App (`web/`):** Vue.js/Vite frontend + Go backend for the web launcher UI.

### Key Architectural Patterns
- **Provider abstraction:** All LLM providers implement a common interface; swappable at runtime
- **Channel adapters:** Each chat platform is a pluggable adapter, decoupled from the agent core
- **Event-driven:** Components communicate via the event bus in `pkg/bus/`
- **Tool/skill registry:** Tools and skills are registered at startup and invoked by the agent loop
- **Config-driven:** Single config file drives provider selection, channel enablement, and feature flags

## Configuration

- `.env.example` — Template for environment variables (LLM API keys, channel tokens)
- `.golangci.yaml` — Linter config (many rules disabled; check before enabling new ones)
- `workspace/` — Default agent workspace and built-in configuration

## Exchange API Pitfalls

### Futures order `amount` is in **contracts**, not base currency

On OKX and Binance, `CreateOrder` for perpetual swaps expects the `amount` parameter to be the **number of contracts**, not the base-currency quantity.

Each market has a `contractSize` field (e.g. CHZ/USDT:USDT on OKX = 10 CHZ per contract). Passing raw base-currency units (e.g. `notionalUSDT / markPrice = 1351 CHZ`) instead of contract count (e.g. `1351 / 10 = 135`) inflates the order size by `contractSize`×, causing OKX error **51008 InsufficientFunds** or Binance equivalent even when the account has sufficient margin.

**Always use `contractsFromNotional(notionalUSD, markPrice, contractSize, minAmount)` in `pkg/tools/futures_helpers.go` to convert a USDT notional to a contract count.** Load the market with `validateActiveSwapMarket` to get `ContractSize` and `Limits.Amount.Min`.

### Futures order `side` vs position `side`

Exchange order APIs use `side = "buy" | "sell"` (order direction), not `"long" | "short"` (position direction). Passing `"short"` as `side` causes OKX error **51000 Parameter side error**.

Use `futuresPositionSide(positionSide string)` in `pkg/tools/futures.go` which maps:
- `"long"` / `"buy"` → order side `"buy"`, position side `"long"`
- `"short"` / `"sell"` → order side `"sell"`, position side `"short"`

The `posSide` / `PositionSide` field in the order request is separate and correctly takes `"long"` / `"short"`.

## Dependencies

Key direct dependencies: `github.com/anthropics/anthropic-sdk-go`, `github.com/openai/openai-go`, `github.com/spf13/cobra`, `github.com/rs/zerolog`, `github.com/modelcontextprotocol/go-sdk`, `modernc.org/sqlite`, `github.com/gorilla/websocket`, plus platform-specific SDKs for each chat channel.

---
> Source: [cryptoquantumwave/khunquant](https://github.com/cryptoquantumwave/khunquant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
