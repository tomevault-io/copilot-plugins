## agent-sdk

> This file provides instructions for AI coding agents working in this repository.

# AGENTS.md - Agent SDK Development Guide

This file provides instructions for AI coding agents working in this repository.

## Project Overview

Agent SDK is a TypeScript library for building AI agents with multi-model support (`createModel` / per-provider factories), MCP integration (`MCPClient`, `MCPAdapter`, optional `mcp_config.json`), skill system, long-term memory via `MemoryManager`, streaming helpers under `src/streaming/`, and JSONL or in-memory session storage.

Optional environment: set `TAVILY_API_KEY` to enable built-in **WebSearch** via the Tavily API (`src/tools/builtin/tavily-search.ts`). Optional tuning: `TAVILY_SEARCH_DEPTH`, `TAVILY_MAX_RESULTS`.

**Zod:** `zod` is a **peerDependency** (^4). It remains in **devDependencies** for this repo’s tests and builds; consumers must install `zod` themselves (see root `README.md` Installation).

## Build/Lint/Test Commands

```bash
# Install dependencies
pnpm install

# Build the project (ESM + CJS + type declarations) via tsup
pnpm build

# Watch mode rebuild during development
pnpm dev

# Type checking (no emit)
pnpm lint

# Run all tests once
pnpm test:run

# Run tests in watch mode
pnpm test

# Run a single test file
pnpm vitest run tests/unit/tools.test.ts

# Run tests matching a pattern
pnpm vitest run -t "should register a tool"

# Clean build artifacts
pnpm clean
```

Requires Node.js >= 18.

## Code Style Guidelines

### Imports

```typescript
// 1. External libraries first
import { z } from 'zod';
import { Command } from 'commander';

// 2. Internal imports - use relative paths with .js extension
import { ToolRegistry } from '../tools/registry.js';
import type { Message, StreamEvent } from '../core/types.js';

// 3. Use `import type` for type-only imports
import type { ModelAdapter, ToolDefinition } from '../core/types.js';
```

### File Structure

Each module should follow this pattern:

```
src/module/
├── index.ts        # Re-exports (barrel file)
├── types.ts        # Type definitions (optional)
├── main-file.ts    # Core implementation
└── helper.ts       # Helper functions
```

### Naming Conventions

- **Files**: `kebab-case.ts` (e.g., `tool-registry.ts`, `stream-transform.ts`)
- **Classes**: `PascalCase` (e.g., `Agent`, `ToolRegistry`, `MCPClient`)
- **Interfaces/Types**: `PascalCase` (e.g., `AgentConfig`, `ToolDefinition`)
- **Functions**: `camelCase` (e.g., `createTool`, `zodToJsonSchema`)
- **Constants**: `UPPER_SNAKE_CASE` (e.g., `DEFAULT_SYSTEM_PROMPT`)
- **Private members**: prefix with `_` for unused, otherwise no prefix

### Type Definitions

```typescript
// Use interfaces for object shapes
export interface AgentConfig {
  model: ModelAdapter;
  systemPrompt?: string;
  tools?: ToolDefinition[];
}

// Use type aliases for unions/intersections
export type StreamEvent = TextEvent | ToolCallEvent | ErrorEvent;

// Export types from dedicated types.ts files
export * from './types.js';
```

### Function Signatures

```typescript
// Async functions that return streams use AsyncIterable
stream(params: ModelParams): AsyncIterable<StreamChunk>

// Factory functions use create* prefix
export function createTool(config: ToolConfig): ToolDefinition
export function createModel(config: CreateModelConfig): ModelAdapter

// Static methods for default instances
export function createAgent(config: AgentConfig): Agent
```

### Error Handling

```typescript
// Always catch and wrap errors with context
try {
  const content = await fs.readFile(path, 'utf-8');
  return { content };
} catch (error) {
  return {
    content: `Error reading file: ${error instanceof Error ? error.message : String(error)}`,
    isError: true
  };
}

// Throw descriptive errors for invalid configuration
if (!this.apiKey) {
  throw new Error('OpenAI API key is required. Set OPENAI_API_KEY environment variable or pass apiKey in config.');
}
```

### Zod Schemas

```typescript
// Use Zod for tool parameter validation (names should match built-in style, e.g. Read / Write)
export const readFileTool = createTool({
  name: 'Read',
  description: 'Read the contents of a file',
  parameters: z.object({
    file_path: z.string().describe('Absolute path to the file to read')
  }),
  handler: async ({ file_path }) => { ... }
});
```

### JSDoc Comments

```typescript
/**
 * Short description of what the function does
 * @param param - Description of parameter
 * @returns Description of return value
 */
export function myFunction(param: string): Result { ... }
```

### Testing

```typescript
import { describe, it, expect } from 'vitest';

describe('FeatureName', () => {
  it('should do something specific', () => {
    const result = myFunction();
    expect(result).toBe(expected);
  });

  it('should handle errors', async () => {
    const result = await riskyOperation();
    expect(result.isError).toBe(true);
  });
});
```

## Architecture Patterns

1. **Factory Functions**: Use `create*` pattern for complex objects (`createTool`, `createModel`, `createMCPAdapter`, etc.)
2. **Interface Segregation**: Keep interfaces focused and composable
3. **Async Iterables**: Use `AsyncIterable<T>` for streaming operations
4. **Adapter Pattern**: Model adapters implement `ModelAdapter`; MCP uses `MCPAdapter` / `MCPClient`
5. **Registry Pattern**: Central registries for tools and skills
6. **Memory**: `MemoryManager` loads optional long-term instructions from CLAUDE.md paths; distinct from session `SessionManager` storage

## Third-party documentation policy

Public docs (`docs/**/*.md`, including `docs/sdk-*.md` and topical guides such as `docs/tool-hook-mechanism.md`) treat **`Agent`** as the only supported integration surface for application code. The repository root `README.md` is for installation and navigation only; factual content lives under `docs/`. Model factories (`createOpenAI`, etc.) configure `AgentConfig.model`; do not document or encourage calling `ModelAdapter.stream` / `complete` directly in product code. Chunk→event helpers (`transformStream`, `toAgentStream`, etc.) are not part of the public API. Keep this aligned with `docs/sdk-overview.md` section 3.

## Module Exports

Each module's `index.ts` should:

```typescript
// Export classes and functions
export { ToolRegistry } from './registry.js';
export { createTool } from './registry.js';

// Export types separately
export type { ToolConfig, ToolResult } from './types.js';
```

The root `src/index.ts` is the main public API; `package.json` also exposes `@ddlqhd/agent-sdk/models` and `@ddlqhd/agent-sdk/tools`.

## Commit Messages

Follow conventional commits:
- `feat:` for new features
- `fix:` for bug fixes
- `docs:` for documentation
- `test:` for tests
- `refactor:` for refactoring

Example: `feat: add MCP integration with stdio transport`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ddlqhd) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
