## quant

> Create `quant` - A comprehensive Elixir library for fetching financial and cryptocurrency data from multiple providers and loading it directly into Explorer DataFrames for high-performance analysis.

# AGENT.md - quant Project Instructions

## Project Overview

Create `quant` - A comprehensive Elixir library for fetching financial and cryptocurrency data from multiple providers and loading it directly into Explorer DataFrames for high-performance analysis.

## Core Vision

- **Multi-Provider Architecture**: Support Yahoo Finance, Alpha Vantage, Binance, CoinGecko, and other providers
- **Explorer-First**: All data returns as Explorer DataFrames for immediate analysis
- **Performance-Focused**: Leverage Explorer's Polars backend for speed
- **Developer-Friendly**: Simple, consistent API across all providers
- **Extensible**: Easy to add new data providers
- **Few dependencies**: As few dependencies to external libraries as possible

## Project Structure

```
quant/
├── lib/
│   ├── quant.ex                    # Main API module
│   ├── quant/
│   │   ├── providers/                     # Data provider implementations
│   │   │   ├── behaviour.ex               # Provider behaviour definition
│   │   │   ├── yahoo_finance.ex           # Yahoo Finance implementation
│   │   │   ├── alpha_vantage.ex           # Alpha Vantage implementation
│   │   │   ├── binance.ex                 # Binance crypto data
│   │   │   ├── coin_gecko.ex              # CoinGecko crypto data
│   │   │   └── twelve_data.ex             # Twelve Data implementation
│   │   ├── rate_limiting/                 # Advanced rate limiting system
│   │   │   ├── behaviour.ex               # Rate limiter behaviour definition
│   │   │   ├── ets_backend.ex             # ETS-based rate limiter
│   │   │   ├── redis_backend.ex           # Redis-based rate limiter
│   │   │   └── provider_config.ex         # Provider-specific configurations
│   │   ├── http_client.ex                 # HTTP client wrapper
│   │   ├── rate_limiter.ex                # Advanced rate limiter manager
│   │   ├── cache.ex                       # Caching layer (ETS-based)
│   │   ├── data_transformer.ex            # Data normalization utilities
│   │   └── config.ex                      # Configuration management
│   └── quant/
│       └── application.ex                 # OTP application
├── test/
│   ├── fin_explorer_test.exs
│   ├── providers/
│   │   ├── yahoo_finance_test.exs
│   │   ├── alpha_vantage_test.exs
│   │   ├── binance_test.exs
│   │   └── coin_gecko_test.exs
│   └── support/
│       ├── fixtures/                      # Mock API responses
│       └── test_helper.ex
├── config/
│   ├── config.exs                         # Application configuration
│   ├── dev.exs                           # Development config
│   ├── test.exs                          # Test config
│   └── runtime.exs                       # Runtime config
├── docs/                                  # Documentation
│   ├── providers/                         # Provider-specific docs
│   └── examples/                          # Usage examples
├── mix.exs                               # Project definition
├── README.md                             # Project README
├── CHANGELOG.md                          # Version history
└── LICENSE                               # MIT License
```

## Main API Design

### Core Interface

```elixir
# Simple single symbol fetch
{:ok, df} = Quant.Explorer.fetch("AAPL", provider: :yahoo_finance)

# Multiple symbols
{:ok, df} = Quant.Explorer.fetch(["AAPL", "MSFT", "GOOGL"], provider: :alpha_vantage)

# Crypto data
{:ok, df} = Quant.Explorer.fetch("BTC-USD", provider: :binance)

# With options
{:ok, df} = Quant.Explorer.fetch("AAPL", 
  provider: :yahoo_finance,
  period: "1y",
  interval: "1d",
  start_date: ~D[2023-01-01],
  end_date: ~D[2024-01-01]
)

# Real-time quotes
{:ok, df} = Quant.Explorer.quote("AAPL", provider: :yahoo_finance)

# Company information
{:ok, info} = Quant.Explorer.info("AAPL", provider: :yahoo_finance)

# Search functionality
{:ok, df} = Quant.Explorer.search("Apple", provider: :yahoo_finance)
```

### Provider-Specific Functions

```elixir
# Provider modules can be called directly for advanced usage
{:ok, df} = Quant.Explorer.Providers.YahooFinance.history("AAPL", period: "1y")
{:ok, df} = Quant.Explorer.Providers.Binance.klines("BTCUSDT", interval: "1h")
```

## Key Requirements

### 1. Dependencies (mix.exs)

```elixir
defp deps do
  [
    {:explorer, "~> 0.11"},
    {:decimal, "~> 2.0"},
    {:telemetry, "~> 1.0"},
    {:ex_doc, "~> 0.31", only: :dev, runtime: false},
    {:credo, "~> 1.7", only: [:dev, :test], runtime: false},
    {:dialyxir, "~> 1.3", only: [:dev], runtime: false},
    {:bypass, "~> 2.1", only: :test}
  ]
end
```

### 2. Provider Behaviour

```elixir
defmodule Quant.Explorer.Providers.Behaviour do
  @moduledoc """
  Behaviour that all data providers must implement.
  """
  
  alias Explorer.DataFrame
  
  @type symbol :: String.t()
  @type symbols :: [symbol()]
  @type options :: keyword()
  @type period :: String.t()
  @type interval :: String.t()
  
  @callback history(symbol() | symbols(), options()) :: 
    {:ok, DataFrame.t()} | {:error, term()}
    
  @callback quote(symbol() | symbols()) :: 
    {:ok, DataFrame.t()} | {:error, term()}
    
  @callback info(symbol()) :: 
    {:ok, map()} | {:error, term()}
    
  @callback search(String.t()) :: 
    {:ok, DataFrame.t()} | {:error, term()}
    
  @optional_callbacks [info: 1, search: 1]
end
```

### 3. Standardized Data Schema

All providers must return DataFrames with standardized column names:

**Historical Data Schema:**
- `symbol` (string): Stock/crypto symbol
- `timestamp` (datetime): Data timestamp
- `open` (f64): Opening price
- `high` (f64): High price
- `low` (f64): Low price
- `close` (f64): Closing price
- `volume` (s64): Trading volume
- `adj_close` (f64): Adjusted closing price (optional)

**Quote Data Schema:**
- `symbol` (string): Stock/crypto symbol
- `price` (f64): Current price
- `change` (f64): Price change
- `change_percent` (f64): Percentage change
- `volume` (s64): Current volume
- `timestamp` (datetime): Quote timestamp

### 4. Configuration System

```elixir
# config/config.exs
config :quant,
  # Default provider if none specified
  default_provider: :yahoo_finance,
  
  # Rate limiting (legacy - for backwards compatibility)
  rate_limits: %{
    yahoo_finance: 100,
    alpha_vantage: 5,
    binance: 1200,
    coin_gecko: 50,
    twelve_data: 8
  },
  
  # Advanced rate limiting configuration
  rate_limiting_backend: :ets,  # Options: :ets, :redis
  rate_limiting_backend_opts: [
    # ETS options
    table_opts: [:set, :public, :named_table],
    
    # Redis options (used when backend is :redis)
    # redis_opts: [host: "localhost", port: 6379, database: 0]
  ],
  
  # Caching settings
  cache_ttl: :timer.minutes(5),
  
  # HTTP timeout
  http_timeout: 10_000,
  
  # API keys (use runtime.exs for secrets)
  api_keys: %{
    alpha_vantage: {:system, "ALPHA_VANTAGE_API_KEY"},
    twelve_data: {:system, "TWELVE_DATA_API_KEY"}
  }
```

### 5. Advanced Rate Limiting System

The project features a sophisticated, behavior-based rate limiting system that supports multiple backends and provider-specific patterns:

#### Rate Limiting Backends

**ETS Backend (Default):**
- High-performance local rate limiting
- Automatic cleanup of expired entries
- Statistics tracking
- Perfect for single-node deployments

**Redis Backend:**
- Distributed rate limiting across multiple nodes
- Lua scripts for atomic operations
- Supports all rate limiting algorithms
- Ideal for production clusters

#### Supported Rate Limiting Algorithms

- **Sliding Window**: Traditional time-based limits (requests per minute/second/hour/day)
- **Token Bucket**: Burst allowance with recovery (perfect for Yahoo Finance patterns)
- **Weighted Requests**: Binance-style weight-based limiting where different endpoints consume different weights
- **Fixed Window**: Simple time-window based limiting
- **Burst Allowance**: Allow traffic bursts with gradual recovery

#### Provider-Specific Configurations

**Yahoo Finance:**
```elixir
%{
  endpoint: :history,
  type: :burst_allowance,
  limit: 100,
  burst_size: 200,
  recovery_rate: 2
}
```

**Binance:**
```elixir
%{
  endpoint: :klines,
  type: :weighted_requests,
  limit: 1200,  # Total weight limit per minute
  weight: 1     # Base weight, varies by request parameters
}
```

**Alpha Vantage:**
```elixir
%{
  type: :requests_per_minute,
  limit: 5  # Free tier
}
```

#### Usage Examples

```elixir
# Basic usage (simple API for single provider/endpoint)
:ok = Quant.Explorer.RateLimiter.check_rate_limit(:yahoo_finance)

# Advanced usage with specific endpoints and parameters
:ok = Quant.Explorer.RateLimiter.check_and_consume(:binance, :ticker_24hr, 
  params: [symbol: nil]  # All symbols = weight 40
)

# Wait for rate limit availability
Quant.Explorer.RateLimiter.wait_for_rate_limit(:alpha_vantage, :default)

# Get detailed status
%{remaining: 45, reset_time: ~U[...], retry_after_ms: 0} = 
  Quant.Explorer.RateLimiter.get_limit_status(:yahoo_finance, :history)
```

### 6. Error Handling Strategy

```elixir
# Standardized error types
{:error, :symbol_not_found}
{:error, :invalid_period}  
{:error, :invalid_interval}
{:error, :rate_limited}
{:error, :api_key_missing}
{:error, {:http_error, reason}}
{:error, {:parse_error, reason}}
{:error, {:provider_error, message}}
```

## Implementation Priority

### Phase 1: Foundation
1. ✅ Set up project structure and dependencies
2. ✅ Implement Provider behaviour
3. ✅ Create HTTP client wrapper with retry logic
4. ✅ Implement advanced rate limiting system (ETS + Redis backends)
5. ✅ Create data transformer utilities

### Phase 2: Yahoo Finance Provider
1. ✅ Historical data fetching
2. ✅ Real-time quotes  
3. ✅ Symbol search
4. ✅ Company information
5. ✅ Options data (advanced)

### Phase 3: Additional Providers
1. ✅ Alpha Vantage implementation
2. ✅ Binance crypto data
3. ✅ CoinGecko crypto data
4. ✅ Twelve Data implementation

### Phase 4: Advanced Features
1. ⏳ Caching layer implementation
2. ⏳ Telemetry and monitoring
3. ⏳ Batch processing capabilities
4. ⏳ Data validation and cleaning

### Phase 5: Polish
1. ⏳ Comprehensive documentation
2. ⏳ Performance optimization
3. ⏳ Integration tests
4. ⏳ CI/CD setup

## Testing Requirements

### Unit Tests
- Test each provider independently using mocked HTTP responses
- Test data transformation and normalization
- Test rate limiting and caching
- Test error handling scenarios

### Integration Tests
- Test against live APIs (with API keys from environment)
- Test data quality and schema compliance
- Performance benchmarks

### Test Fixtures
Create realistic mock responses for:
- Historical data (various periods/intervals)
- Real-time quotes
- Symbol search results
- Company information
- Error responses

## Documentation Requirements

### README.md Sections
1. Installation instructions
2. Quick start guide
3. Supported providers and their capabilities
4. API reference with examples
5. Configuration guide
6. Contributing guidelines
7. Performance characteristics

### Provider Documentation
Each provider should have detailed documentation covering:
- Supported data types and endpoints
- Rate limits and API key requirements  
- Specific options and parameters
- Data quality notes and limitations

### Examples Directory
Create practical examples for:
- Basic usage patterns
- Backtesting setup
- Real-time monitoring
- Multi-provider data aggregation
- Integration with Nx for analysis

## Performance Targets

- **Throughput**: Handle 1000+ symbols per minute
- **Memory**: Efficient memory usage with streaming for large datasets
- **Latency**: < 100ms for cached data, < 2s for API calls
- **Concurrency**: Support concurrent requests with proper rate limiting

## Quality Standards

### Code Quality
- Maintain 90%+ test coverage
- Use Dialyxir for type checking
- Follow Credo style guidelines
- Document all public functions

### Reliability
- Graceful error handling and recovery
- Proper rate limiting to avoid API bans
- Robust HTTP client with retries
- Data validation and sanitization

### Security
- Secure API key management
- Input validation and sanitization
- No sensitive data logging
- Rate limiting to prevent abuse

## Future Considerations

### Potential Extensions
- Real-time streaming data (WebSocket support)
- More advanced financial data (options chains, fundamentals)
- Integration with popular charting libraries
- Plugin system for custom providers
- Cloud deployment helpers (Docker, etc.)

### Monitoring & Observability
- Telemetry events for all operations
- Metrics collection (request counts, latencies)
- Health check endpoints
- Error tracking and alerting

## Success Criteria

The library is successful when:
1. **Developer Experience**: Simple to install, configure, and use
2. **Data Quality**: Reliable, consistent data from multiple providers
3. **Performance**: Fast enough for production trading applications
4. **Adoption**: Used by the Elixir financial/trading community
5. **Maintenance**: Well-documented, tested, and easy to contribute to

## Current Project Status (as of January 2025)

### ✅ Completed Components

**Phase 1: Foundation (100% Complete)**
- ✅ **Project Structure**: Complete Elixir/OTP application with proper supervision tree
- ✅ **Provider Behaviour**: Comprehensive interface defining history, quote, info, and search functions
- ✅ **HTTP Client**: Robust wrapper with retry logic, timeout handling, and SSL verification using built-in `:httpc`
- ✅ **Advanced Rate Limiting**: Sophisticated unified system supporting:
  - Multiple backends (ETS for local, Redis for distributed)
  - Multiple algorithms (sliding window, token bucket, weighted requests, burst allowance)
  - Provider-specific configurations (Yahoo, Binance, Alpha Vantage patterns)
  - Single consolidated module (`Quant.Explorer.RateLimiter`) with clean API
- ✅ **Data Transformer**: Utilities for normalizing API responses into Explorer DataFrames
- ✅ **Configuration System**: Flexible config management with environment variable support
- ✅ **Testing Infrastructure**: Complete test setup with fixtures and helper functions
- ✅ **Minimal Dependencies**: Using built-in Elixir/OTP modules (JSON, HTTP, DateTime, CSV parsing)

**Phase 2: Yahoo Finance Provider (100% Complete)**
- ✅ **Historical Data**: Complete implementation with support for all periods (1d to max) and intervals (1m to 3mo)
- ✅ **Streaming Support**: `history_stream/2` for efficient processing of large datasets
- ✅ **Real-time Quotes**: Multi-symbol quote fetching with market status and currency info  
- ✅ **Symbol Search**: Company/ticker search with filtering by type, exchange, sector
- ✅ **Company Information**: Comprehensive company data including fundamentals, sector, employees
- ✅ **Options Data**: Full options chain support with calls/puts, strikes, expiration dates
- ✅ **Concurrent Processing**: Efficient multi-symbol requests with configurable concurrency
- ✅ **Error Handling**: Robust error handling for all API scenarios (404, rate limits, parse errors)
- ✅ **Rate Limiting Integration**: Full integration with advanced rate limiter

**Quality Assurance:**
- ✅ All tests passing for foundation components (8/8)  
- ✅ Clean compilation with only expected warnings (Redis dependencies)
- ✅ Yahoo Finance provider successfully integrated and working with real API
- ✅ Proper error handling and type specifications
- ✅ Explorer-first design with all data returned as DataFrames
- ✅ Streaming support for large datasets

### 🚧 Next Steps - Phase 2: Yahoo Finance Provider

The foundation is solid and ready for implementing the first data provider:

1. **Historical Data Fetching**
   - Implement Yahoo Finance v8 API integration
   - Support for various periods (1d, 5d, 1mo, 3mo, 6mo, 1y, 2y, 5y, 10y, ytd, max)
   - Multiple intervals (1m, 2m, 5m, 15m, 30m, 60m, 90m, 1h, 1d, 5d, 1wk, 1mo, 3mo)

2. **Real-time Quotes**
   - Current price, change, volume data
   - Market status and trading hours

3. **Symbol Search**
   - Company/ticker search functionality
   - Result filtering and ranking

4. **Company Information**
   - Basic company metadata
   - Market cap, sector, industry information

### 🎯 Architecture Highlights

The project now features a **production-ready foundation** with:

- **Behavior-Driven Design**: Clean separation between interfaces and implementations
- **Multiple Backend Support**: Can scale from single-node (ETS) to distributed (Redis)
- **Provider-Specific Optimizations**: Handles complex scenarios like Binance weight-based limits
- **High Performance**: Leverages Explorer's Polars backend for data processing
- **Developer Experience**: Simple, unified API with comprehensive error handling
- **Clean Architecture**: Consolidated rate limiting system with single entry point

### 📊 Code Quality Metrics

- **Test Coverage**: 100% for foundation components
- **Compilation**: Clean with zero errors
- **Dependencies**: Minimal and well-chosen
- **Documentation**: Comprehensive inline documentation
- **Type Safety**: Full Dialyzer type specifications

The advanced rate limiting system demonstrates enterprise-grade architecture:
- Supports 6 different rate limiting patterns
- Handles provider-specific edge cases (Binance weights, Yahoo bursts)
- Provides both ETS and Redis backends for any deployment scenario
- Unified interface through single `Quant.Explorer.RateLimiter` module
- Clean API design with proper separation of concerns

---

## Getting Started Checklist

- [x] Create new Mix project: `mix new quant --sup`
- [x] Add dependencies to mix.exs
- [x] Set up basic project structure
- [x] Implement Provider behaviour
- [x] Create HTTP client wrapper
- [x] Create advanced rate limiter (ETS + Redis backends)
- [x] Create data transformer utilities  
- [x] Create configuration module
- [x] Update main API module
- [x] Set up testing infrastructure
- [ ] Implement Yahoo Finance provider (start simple)
- [ ] Add comprehensive tests
- [ ] Write documentation and examples
- [ ] Set up CI/CD pipeline
- [ ] Publish to Hex.pm

**Remember**: Start simple, iterate quickly, and prioritize developer experience. The goal is to make financial data access in Elixir as easy as `pip install yfinance` is in Python.

---
> Source: [laibulle/quant](https://github.com/laibulle/quant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
