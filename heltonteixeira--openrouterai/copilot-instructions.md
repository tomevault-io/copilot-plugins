## openrouterai

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Error Prevention and Learning

### Using SOLUTIONS.md as Reference
- **SOLUTIONS.md** contains documented mistakes and their resolutions to prevent recurrence
- **Always consult SOLUTIONS.md** before implementing changes to avoid past errors
- **Quick Reference Section** provides critical patterns: DXT properties, platform identifiers, CLI commands
- **Core Standards** section outlines established patterns for manifest structure, TypeScript, and environment handling
- **Error Resolution Process** provides step-by-step validation workflow

### Error Prevention Workflow
1. **Check SOLUTIONS.md first** - Review quick reference and relevant error patterns
2. **Validate Early** - Use `dxt validate manifest.json` before any changes
3. **Follow Established Patterns** - Use documented solutions from similar past issues
4. **Update SOLUTIONS.md** - Document new errors and their solutions for future reference

## Essential Commands

### Build & Development
- `pnpm run build` - TypeScript compilation and makes dist/*.js executable
- `pnpm run watch` - Development with live TypeScript watching
- `pnpm run prepare` - Pre-publish build (runs automatically on npm install)

### Testing
- No test framework currently configured
- Manual testing via MCP client integration

### Package Management
- Uses `pnpm` for package management
- ES modules with Node.js 18+ required
- Entry point: `dist/index.js` (built from `src/index.ts`)

## Architecture Overview

### Core MCP Server Structure
- **Entry Point**: `src/index.ts` - Main server initialization with environment variable parsing
- **Tool Registration**: `src/tool-handlers.ts` - Central dispatcher for all MCP tools with comprehensive input schemas
- **API Layer**: `src/openrouter-api.ts` - OpenRouter.ai API client with rate limiting and retry logic
- **Caching**: `src/model-cache.ts` - Singleton model cache with 1-hour expiry
- **Types**: `src/types.ts` - Unified `ToolResult` interface for all tool responses

### Tool Handler Pattern
Individual tool handlers in `src/tool-handlers/`:
- `chat-completion.ts` - OpenAI-compatible chat completions with provider routing
- `search-models.ts` - Model discovery with filtering capabilities
- `get-model-info.ts` - Single model metadata retrieval
- `validate-model.ts` - Model ID validation

### Response Architecture
All tools return standardized `ToolResult` structure:
```typescript
interface ToolResult {
  isError: boolean;
  content: Array<{ type: "text"; text: string }>;
}
```

## Key Technical Patterns

### Environment Configuration
The server reads extensive environment variables for defaults:
- `OPENROUTER_API_KEY` (required)
- `OPENROUTER_DEFAULT_MODEL` 
- `OPENROUTER_MAX_TOKENS`
- Phase 1: `OPENROUTER_PROVIDER_QUANTIZATIONS`, `OPENROUTER_PROVIDER_IGNORE`
- Phase 2: `OPENROUTER_PROVIDER_SORT`, `OPENROUTER_PROVIDER_ORDER`, etc.

### Error Handling Strategy
- All errors logged to stderr for Claude Desktop visibility
- Structured error responses with `isError: true`
- Rate limit detection with automatic retry and exponential backoff
- Try-catch wrapper in main tool handler dispatcher

### Provider Routing System
Two-phase provider routing controls:
- **Phase 1**: Basic filtering (quantizations, ignore lists)
- **Phase 2**: Advanced routing (sort order, parameter requirements, data collection policies)

### Caching Implementation
- Singleton `ModelCache` class with 1-hour model data expiry
- Memory-efficient storage of OpenRouter model registry
- Automatic cache invalidation and refresh

## Development Constraints

### TypeScript Configuration
- ES2022 target with NodeNext modules
- Strict mode enabled
- Declaration files generated in dist/
- Must use `.js` extensions in imports for ES modules

### Dependencies
- `@modelcontextprotocol/sdk` - Core MCP implementation
- `openai` - Chat completions via OpenRouter
- `axios` - HTTP client for model registry API
- `typescript` - Development dependency

### OpenRouter Integration
- Base URL: `https://openrouter.ai/api/v1`
- Required headers: `HTTP-Referer`, `X-Title`
- Rate limit headers: `x-ratelimit-remaining`, `x-ratelimit-reset`
- Model registry endpoint: `/models`

---
> Source: [heltonteixeira/openrouterai](https://github.com/heltonteixeira/openrouterai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
