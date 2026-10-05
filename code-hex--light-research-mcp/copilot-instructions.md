## light-research-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Package Manager

This project uses **pnpm** as the package manager. All npm commands should be replaced with pnpm equivalents:
- `pnpm install` (not npm install)
- `pnpm build` (not npm run build)
- `pnpm test` (not npm test)

## Essential Commands

### Development Workflow
```bash
# Install dependencies
pnpm install

# Build the project (required before running)
pnpm build

# Build in watch mode for development
pnpm dev

# Install Playwright browsers (required for content extraction)
pnpm install-browsers

# Run the CLI tool
node dist/bin/llmresearcher.js [query]
# or if globally linked: llmresearcher [query]
```

### Testing
```bash
# Run tests once (CI mode) - ALWAYS USE THIS
pnpm test:run

# Run specific test file
pnpm test:run search.test.ts

# Run tests in watch mode (only for development)
pnpm test

# Run tests with coverage
pnpm test -- --coverage

# Type checking without compilation
pnpm type-check
```

**Important: Always use `pnpm test:run` instead of `pnpm test` for running tests. This ensures consistent, non-interactive test execution.**

### Build & Clean
```bash
# Clean build artifacts
pnpm clean

# Build for production
pnpm build
```

## Architecture Overview

This is a TypeScript CLI application that implements a lightweight MCP (Model Context Protocol) server for LLM orchestration. The architecture follows a modular design with clear separation of concerns:

### Core Architecture Pattern

The application uses a **composition-based architecture** where the main `LLMResearcher` class orchestrates three specialized components:

1. **DuckDuckGoSearcher** (`src/search.ts`) - Handles web search via DuckDuckGo HTML scraping
2. **ContentExtractor** (`src/extractor.ts`) - Extracts and processes web page content using Playwright
3. **CLIInterface** (`src/cli.ts`) - Manages user interaction and command handling

### Data Flow Pipeline

```
User Query → DuckDuckGo Search → URL Selection → Playwright Extraction → Content Processing → Markdown Output
```

The content processing pipeline implements strict sanitization:
- **Input**: Raw HTML from Playwright-rendered pages
- **Processing**: @mozilla/readability → DOMPurify sanitization → Turndown markdown conversion
- **Output**: Clean markdown with only h1-h3, bold, italic, and links

### Configuration System

The application uses a layered configuration approach:
1. **Environment variables** (`.env` file)
2. **RC file** (`~/.llmresearcherrc`) for user preferences
3. **Default values** with runtime overrides

Configuration is centralized in `src/config.ts` and accessed globally through the `config` object.

### CLI Architecture

The CLI supports three operational modes:
- **Search mode**: `llmresearcher "query"` - Searches and allows interactive result browsing
- **Direct URL mode**: `llmresearcher -u https://example.com` - Directly extracts content from a URL
- **Interactive mode**: `llmresearcher` - Enters a persistent interactive session

### Rate Limiting & Error Handling

The application implements robust rate limiting (1 req/sec for DuckDuckGo) with exponential backoff retry logic. All network operations include comprehensive error handling with graceful degradation.

### Build System

- **tsup**: Fast TypeScript bundler that creates ESM output in `dist/`
- **Entry points**: `src/bin.ts` (CLI) and `src/index.ts` (library)
- **Output**: `dist/bin/llmresearcher.js` (executable) and library modules

### Testing Strategy

The test suite uses **vitest** with four test categories:
- **Unit tests**: Individual component testing (`search.test.ts`, `extractor.test.ts`, `config.test.ts`)
- **Integration tests**: End-to-end workflow testing (`integration.test.ts`)

Tests require network access for DuckDuckGo and Playwright functionality, with timeouts configured for external dependencies.

## Key Implementation Details

### TypeScript Configuration
- Strict type checking enabled with `noUncheckedIndexedAccess`
- ESM modules with `.js` imports (TypeScript convention for ESM)
- Path mapping: `@/*` aliases to `src/*`

### Browser Automation
- Uses Playwright's bundled Chromium (no local Chrome required)
- Resource blocking for performance: blocks images, CSS, fonts during extraction
- Headless operation with configurable user agent

### Content Sanitization
- Three-stage sanitization: Readability → DOMPurify → Turndown
- Strict allowlist: only h1-h3, strong, em, a tags preserved
- Fallback extraction strategy if Readability fails

### Error Handling Patterns
- Typed error handling with proper Error casting
- Graceful resource cleanup in all exit paths
- Browser resource management with automatic cleanup

---
> Source: [Code-Hex/light-research-mcp](https://github.com/Code-Hex/light-research-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-05 -->
