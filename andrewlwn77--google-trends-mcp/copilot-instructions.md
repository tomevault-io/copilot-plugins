## google-trends-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Google Trends MCP (Model Context Protocol) server that provides access to Google Trends data through the RapidAPI Google Trends Scraper API. The server implements three main endpoints:

1. **Status**: Check API health and status
2. **Questions**: Get trending questions from Google Trends
3. **Trending**: Get currently trending topics from Google Trends

## Commands

### Development
```bash
# Install dependencies
npm install

# Build TypeScript
npm run build

# Run in development mode with auto-reload
npm run dev

# Run the compiled server
npm start
```

### Testing
```bash
# Run all tests
npm test

# Run specific test suites
npm run test:unit          # Unit tests only
npm run test:integration   # Integration tests only
npm run test:watch        # Watch mode

# Run tests with coverage
npm test -- --coverage

# Run a single test file
npm test -- tests/integration/search.test.ts

# Run tests matching a pattern
npm test -- --testNamePattern="should search for"

# Run tests with single worker (useful for debugging)
npm test -- --maxWorkers=1
```

## Architecture

### MCP Server Structure
The server is built using the Model Context Protocol SDK and follows this pattern:

1. **Main Server Class** (`src/index.ts`): `GoogleTrendsMcpServer` encapsulates all functionality
   - Uses `StdioServerTransport` for communication
   - Registers handlers for `ListToolsRequestSchema` and `CallToolRequestSchema`
   - Each tool handler makes API calls and returns formatted responses

2. **API Integration Pattern**:
   - `makeApiRequest()`: Centralized method for all API calls with consistent headers
   - Each handler (status, questions, trending) follows the same pattern:
     - Make API request to Google Trends API
     - Normalize response to match TypeScript types
     - Return formatted MCP response

3. **Type System** (`src/types.ts`):
   - Separate interfaces for parameters (`StatusParams`, `QuestionsParams`, `TrendingParams`)
   - Response types for Google Trends data structures
   - Flexible types to accommodate API response variations

### Testing Architecture
Tests use a parallel structure that mirrors the production code:

1. **Test Client** (`tests/helpers/test-server.ts`): 
   - `GoogleTrendsTestClient` class duplicates the API logic without MCP protocol
   - Allows direct testing of API integration
   - Shares the same request logic as the main server

2. **Integration Tests**: Each endpoint has comprehensive tests covering:
   - Happy path API calls
   - Parameter handling (language, country)
   - API response structure validation

### Key Architectural Decisions

1. **Simple API Integration**: Direct API calls to Google Trends endpoints
   - `/status` - Check API health
   - `/questions` - Get trending questions
   - `/trending` - Get trending topics

2. **Error Handling**: Two-layer approach:
   - API errors are caught and returned as formatted error responses
   - MCP protocol errors are handled by the SDK

3. **Environment Configuration**: 
   - API key via `GOOGLE_TRENDS_API_KEY` env variable
   - Falls back to example key if not provided

## API Response Structure

The Google Trends API returns data in these formats:

- **Status**: Simple status object with API health information
- **Questions**: Array of trending question objects
- **Trending**: Array of trending topic objects

## Running the MCP Server

To test the server with MCP protocol:
```bash
# Build first
npm run build

# Or run directly
npm start

# For development with auto-reload
npm run dev
```

The server communicates via stdio and implements the standard MCP protocol for tool discovery and execution.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andrewlwn77) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
