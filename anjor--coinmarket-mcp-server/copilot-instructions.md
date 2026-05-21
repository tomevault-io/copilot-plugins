## coinmarket-mcp-server

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Coinmarket MCP (Model Context Protocol) Server that provides cryptocurrency data from the CoinMarketCap API. The server implements the MCP specification to allow AI assistants to access cryptocurrency listings and quotes.

## Architecture

- **MCP Server**: Built using the `mcp` Python library, communicates via stdin/stdout
- **API Integration**: Interfaces with CoinMarketCap Pro API for cryptocurrency data
- **Resource System**: Exposes cryptocurrency data as resources with `coinmarket://` URI scheme
- **Tool System**: Provides two tools for getting listings and quotes

### Key Components

- `src/coinmarket_service/server.py`: Main MCP server implementation with:
  - Resource handlers for cryptocurrency listings and quotes
  - Tool handlers for `get_currency_listings` and `get_quotes`
  - API key authentication and request handling
- `src/coinmarket_service/__init__.py`: Package entry point that runs the async server

## Development Commands

### Setup and Installation
```bash
# Install uv package manager (if not already installed)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install dependencies
uv sync

# Run the server
uv run coinmarket_service
```

### Docker
```bash
# Build Docker image
docker build -t coinmarket-service .

# Run with Docker
docker run -e COINMARKET_API_KEY=your_api_key_here coinmarket-service
```

### Linting
```bash
# Run ruff linter
uv run ruff check
uv run ruff format
```

## Configuration

- Requires `COINMARKET_API_KEY` environment variable
- Uses `.env` file support via python-dotenv
- Configuration for Claude Desktop requires absolute path to repository

## MCP Server Details

The server implements:
- **Resources**: 
  - `coinmarket://cryptocurrency/listings` - Latest cryptocurrency listings
  - `coinmarket://cryptocurrency/quotes` - Cryptocurrency quotes (supports slug/symbol query params)
- **Tools**:
  - `get_currency_listings` - No parameters required
  - `get_quotes` - Optional `slug` or `symbol` parameters

## API Integration

- Uses CoinMarketCap Pro API v1
- Implements rate limiting and error handling
- Returns JSON formatted data
- Supports both slug-based and symbol-based cryptocurrency queries

---
> Source: [anjor/coinmarket-mcp-server](https://github.com/anjor/coinmarket-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
