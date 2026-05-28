## tradingview-ws-client-old

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Building and Running
- `go run cmd/app/main.go` - Run the main application server
- `go run cmd/example/main.go` - Run the example client
- `go build -o bin/app cmd/app/main.go` - Build the main application
- `go test ./...` - Run all tests
- `go test pkg/tvwsclient/...` - Run tests for the TradingView client package

### Code Generation
- `go generate ./ent` - Generate Ent database schema code (run after modifying schema files in `ent/schema/`)

## Architecture Overview

### Core Components

**TradingView WebSocket Client (`pkg/tvwsclient/`)**
- Real-time WebSocket client for TradingView data streams
- Handles authentication, heartbeat, reconnection, and message parsing
- Key files: `client.go`, `auth_token.go`, `types.go`, `send_message.go`

**Database Layer (`ent/`)**
- Uses Facebook Ent ORM for PostgreSQL data persistence
- Enhanced schema with 6 entities: `Symbol`, `Candle`, `ActiveSession`, `StudySession`, `Indicator`, `IndicatorData`
- Comprehensive relationships and indexing for performance optimization
- Schema definitions in `ent/schema/`, generated code in other `ent/` files
- Key entities:
  - `Symbol`: Comprehensive symbol metadata with exchange, type, sector, market cap
  - `Candle`: OHLCV data with quality indicators and advanced market microstructure fields
  - `ActiveSession`: Real-time subscription tracking with status and error handling
  - `StudySession`: Technical analysis sessions for indicator calculations
  - `Indicator`: Individual technical indicators (RSI, MACD, BB, etc.) with parameters
  - `IndicatorData`: Time-series calculated indicator values with quality tracking

**Service Layer (`internal/service/`)**
- `TradingViewService` orchestrates WebSocket client and database operations
- Processes real-time messages (quotes, candle updates) and persists to database
- Manages subscription lifecycle and client reconnection

**HTTP API (`internal/handler/`)**
- Fiber-based REST API for managing subscriptions and querying data
- Endpoints for symbols, candles, quotes, health checks, and technical indicators
- Standardized API response format using DTOs (`internal/dto/`)
- Consistent error handling, pagination, and response metadata
- Response DTOs include: `SymbolResponse`, `CandleResponse`, `ActiveSessionResponse`, `StudySessionResponse`, `IndicatorResponse`

**Configuration (`internal/config/`)**
- Advanced configuration system supporting YAML files, environment variables, and CLI flags
- Multiple configuration methods with priority: CLI flags > Environment variables > Config file > Defaults
- Required: `TRADINGVIEW_DEVICE_TOKEN`, `TRADINGVIEW_SESSION_ID`, `TRADINGVIEW_SESSION_SIGN`
- Database connection via `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`, `DB_SSLMODE`
- Use `make test-config` to validate configuration without connecting to database

**Technical Indicators Framework (`pkg/tvwsclient/indicators.go`)**
- Comprehensive technical analysis framework supporting 10+ indicators
- Supported indicators: RSI, MACD, Bollinger Bands, SMA, EMA, WMA, Stochastic, Williams %R, CCI, Momentum, ROC, ADX, ATR
- Study session management for grouping related indicators
- Real-time calculation with candle data integration
- REST API endpoints for indicator CRUD operations (`internal/handler/indicators.go`)

### Data Flow
1. Application starts → loads config → connects to PostgreSQL → initializes TradingView WebSocket client
2. HTTP requests create/delete subscriptions → updates `ActiveSession` table → sends WebSocket subscribe/unsubscribe messages
3. Real-time data arrives via WebSocket → processed by service layer → quotes cached in Ristretto, candles persisted to PostgreSQL
4. API queries return cached quotes or historical candles from database

### Key Patterns
- Uses structured logging with `slog` throughout
- Ristretto cache for real-time quote data
- Concurrent message processing with goroutines and channels
- Automatic reconnection and session restoration on WebSocket failures
- Database migrations handled automatically by Ent on startup

---
> Source: [iiiyu/tradingview-ws-client-old](https://github.com/iiiyu/tradingview-ws-client-old) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
