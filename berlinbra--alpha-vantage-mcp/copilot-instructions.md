## alpha-vantage-mcp

> The Alpha Vantage MCP Server is a Model Context Protocol (MCP) implementation that provides real-time access to financial market data through the Alpha Vantage API. This server enables Claude and other AI assistants to retrieve stock quotes, company information, and cryptocurrency data.

# Alpha Vantage MCP Server - Claude.md

## Overview

The Alpha Vantage MCP Server is a Model Context Protocol (MCP) implementation that provides real-time access to financial market data through the Alpha Vantage API. This server enables Claude and other AI assistants to retrieve stock quotes, company information, and cryptocurrency data.

## Architecture

### Core Components

- **server.py** - Main MCP server implementation that handles tool registration and execution
- **tools.py** - Utility functions for API requests and data formatting
- **smithery.yaml** - Smithery MCP configuration for automated deployment

### Key Features

1. **Stock Market Data**
   - Real-time stock quotes with price, volume, and change data
   - Detailed company information (sector, industry, market cap)
   - Historical daily time series data (OHLCV)

2. **Cryptocurrency Support** 
   - Real-time cryptocurrency exchange rates with bid/ask prices
   - Daily, weekly, and monthly cryptocurrency time series data

3. **Advanced Options Data**
   - Historical options chain data with Greeks calculations
   - Advanced filtering and sorting capabilities by strike, volume, IV, etc.

4. **Error Handling & Rate Limiting**
   - Built-in error handling for API failures
   - Rate limit management and timeout handling
   - Comprehensive error messages
   - **CRITICAL**: Never use placeholder data in responses - always handle errors properly and transparently

## Available Tools

The server implements 12 MCP tools:

1. **get-stock-quote** - Current stock quote information
2. **get-company-info** - Detailed company overview data
3. **get-crypto-exchange-rate** - Cryptocurrency exchange rates
4. **get-time-series** - Historical daily stock price data
5. **get-realtime-options** - Real-time options chain data (premium required)
6. **get-historical-options** - Options chain data with sorting
7. **get-etf-profile** - Comprehensive ETF profile with holdings and sector allocation
8. **get-crypto-daily** - Daily cryptocurrency time series
9. **get-crypto-weekly** - Weekly cryptocurrency time series
10. **get-crypto-monthly** - Monthly cryptocurrency time series
11. **get-earnings-calendar** - Upcoming earnings calendar data
12. **get-historical-earnings** - Historical earnings data

## Smithery MCP Integration

### Configuration (smithery.yaml:3-18)

The repository uses Smithery for automated MCP deployment and configuration:

- **startCommand**: Defines STDIO-based communication
- **configSchema**: JSON Schema requiring `alphaVantageApiKey`
- **commandFunction**: JavaScript function that generates the CLI command with proper environment variables

### Key Smithery Features

- **Automated Installation**: One-command install via `npx @smithery/cli install @berlinbra/alpha-vantage-mcp --client claude`
- **Configuration Management**: Handles API key injection and environment setup
- **Claude Desktop Integration**: Direct integration with Claude desktop client

### Smithery Benefits

1. **Simplified Deployment**: No manual configuration of `claude_desktop_config.json`
2. **Environment Management**: Automatic API key handling
3. **Version Management**: Centralized distribution and updates
4. **Cross-Platform Support**: Works on macOS, Windows, and Linux

## Installation Methods

### 1. Via Smithery (Recommended)
```bash
npx -y @smithery/cli install @berlinbra/alpha-vantage-mcp --client claude
```

### 2. Via Docker
- Build local image and configure `claude_desktop_config.json`
- Requires manual environment variable setup

### 3. Development Setup
- Use `uv` package manager for local development
- Requires manual path configuration

## API Integration

### Alpha Vantage API (tools.py:12-70)

- **Base URL**: `https://www.alphavantage.co/query`
- **Authentication**: API key via `ALPHA_VANTAGE_API_KEY` environment variable
- **Request Handling**: Async HTTP client with 30-second timeout
- **Error Management**: Comprehensive error handling for rate limits, authentication, and network issues

### Supported API Functions

**Free Tier Compatible:**
- `GLOBAL_QUOTE` - Stock quotes
- `OVERVIEW` - Company information  
- `CURRENCY_EXCHANGE_RATE` - Crypto exchange rates
- `TIME_SERIES_DAILY` - Historical stock data
- `HISTORICAL_OPTIONS` - Options chain data
- `ETF_PROFILE` - ETF profile data with holdings and sectors
- `EARNINGS_CALENDAR` - Upcoming earnings data
- `EARNINGS` - Historical earnings data
- `DIGITAL_CURRENCY_DAILY/WEEKLY/MONTHLY` - Crypto time series

**Premium Subscription Required:**
- `REALTIME_OPTIONS` - Real-time options chain data (returns demo data with free API keys)

## Development Commands

### Running the Server
```bash
# Direct execution
uv run src/alpha_vantage_mcp/server.py

# With MCP Inspector
npx @modelcontextprotocol/inspector uv --directory /path/to/alpha-vantage-mcp run src/alpha_vantage_mcp/server.py
```

### Package Management
```bash
uv install -e .
```

## Dependencies

- **Python 3.12+** - Required runtime
- **httpx** - Async HTTP client for API requests
- **mcp** - Model Context Protocol framework

## Contributors

- [berlinbra](https://github.com/berlinbra) - Primary maintainer
- [zzulanas](https://github.com/zzulanas) - Contributor

## Error Handling Guidelines

### Core Principles

1. **No Placeholder Data**: Under no circumstances should the server return placeholder or demo data as if it were real market data
2. **Transparent Error Reporting**: All API errors, rate limits, and access issues must be clearly communicated to users
3. **Proper Error Detection**: The server must detect and flag demo/placeholder responses from the API
4. **User Education**: Error messages should help users understand API limitations and subscription requirements

### Common Error Scenarios

- **Rate Limiting**: Alpha Vantage free tier has request limits
- **Premium Features**: Some endpoints (like realtime options) require paid subscriptions
- **API Key Issues**: Invalid or expired API keys
- **Demo Data**: Alpha Vantage may return placeholder data for demo purposes
- **Network Issues**: Connection timeouts and service unavailability

### Implementation Requirements

All formatting functions in `tools.py` must:
- Detect placeholder/demo data patterns (e.g., "XXYYZZ" symbols, "2099-99-99" dates)
- Return clear error messages instead of formatting fake data
- Provide guidance on resolving access issues
- Log error patterns for debugging

## License

MIT License - Free for use, modification, and distribution

---
> Source: [berlinbra/alpha-vantage-mcp](https://github.com/berlinbra/alpha-vantage-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
