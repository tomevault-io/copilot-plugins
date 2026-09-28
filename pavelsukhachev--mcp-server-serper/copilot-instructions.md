## mcp-server-serper

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an MCP (Model Context Protocol) server that integrates with the Serper API to provide advanced Google search capabilities with full support for search operators. The server is optimized for complex queries including LinkedIn profile/company searches, technical documentation, research papers, and more.

## Development Commands

```bash
# Build the project
npm run build

# Run tests
npm test                    # Run all tests
npm run test:watch         # Run tests in watch mode
npm run test:coverage      # Run tests with coverage
npm run test:integration   # Run integration tests only

# Development
npm run watch              # Continuous TypeScript compilation
npm run inspector          # Launch MCP Inspector for debugging
```

## Architecture

The codebase follows a clean service-oriented architecture:

- **Entry Point**: `src/index.ts` sets up the MCP server using stdio transport
- **Service Layer**: `src/services/serper-client.ts` handles all Serper API interactions with advanced query building
- **Tools Layer**: `src/tools/search-tool.ts` wraps search and scrape operations for MCP
- **Type Definitions**: `src/types/serper.ts` provides TypeScript interfaces for API contracts
- **Prompts**: `src/prompts/index.ts` contains pre-configured research prompts

## Key Implementation Details

### Search Query Building
The SerperClient builds complex Google search queries supporting:
- Advanced operators (site:, filetype:, intitle:, inurl:, intext:)
- Date filtering (before:/after: operators)
- Exact phrase matching with quotes
- Exclusion with minus operator
- OR operator for alternatives
- Parentheses for grouping
- LinkedIn-specific searches:
  - People: site:linkedin.com/in/ "name" "title" "location"
  - Companies: site:linkedin.com/company/ "industry" "keyword"
- Number ranges with #..# operator
- Wildcard matching with *
- Related sites with related: operator

### API Integration
- Requires `SERPER_API_KEY` environment variable
- Uses fetch API for HTTP requests
- Handles both search and scrape endpoints
- Returns structured results including organic results, knowledge graph, and related searches

### Testing Strategy
- Unit tests mock the Serper API responses
- Integration tests (`*.integration.test.ts`) require actual API key
- Tests cover query building, error handling, and response parsing

## Environment Setup

1. Create `.env` file with `SERPER_API_KEY`
2. Install dependencies with `npm install`
3. Build before running: `npm run build`

## MCP Tools Exposed

- `google_search`: Performs advanced Google searches with full operator support
  - Required: query (q)
  - Optional: region (gl, default: 'us'), language (hl, default: 'en')
  - Supports all Google advanced operators in the query
  - Examples:
    - LinkedIn people: `q: "site:linkedin.com/in/ \"data scientist\" Python San Francisco"`
    - LinkedIn companies: `q: "site:linkedin.com/company/ fintech \"series B\""`
    - Technical: `q: "site:stackoverflow.com TypeError pandas -duplicate"`
    - Research: `q: "filetype:pdf \"machine learning\" healthcare after:2023-01-01"`
- `scrape`: Extracts content from web pages in text/markdown format

## MCP Prompts Available

- `google-search-guide`: Complete guide to Google advanced search operators
- `smart-search`: Helps craft optimized search queries based on intent
- `research-topic`: Comprehensive research using advanced search techniques
- `compare-sources`: Compare information from multiple sources
- `fact-check`: Verify claims across authoritative sources
- `technical-search`: Focused technical and programming searches

---
> Source: [pavelsukhachev/mcp-server-serper](https://github.com/pavelsukhachev/mcp-server-serper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-27 -->
