## ynab-mcp-server

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

YNAB MCP Server - A Model Context Protocol server providing AI assistants with comprehensive access to YNAB (You Need A Budget) data. Features 27+ tools for budget management, transactions, categories, payees, scheduled transactions, transfers, and automated budget recommendations.

## Essential Commands

```bash
# Development
npm run dev          # Start with hot reload using tsx
npm run build        # Compile TypeScript to dist/
npm start            # Run production build

# Testing
npm test             # Run vitest
npm run test:watch   # Run tests in watch mode
npm run test:coverage # Run with coverage

# Other
npm run lint         # TypeScript type checking (tsc --noEmit)
npm run clean        # Remove dist/ directory
```

## Architecture

### Core Data Flow
```
MCP Client → StdioTransport → YnabMcpServer → Tool Registry → YNABClient → YNAB API
```

### Key Components

**Entry Point & Server** (`src/index.ts`, `src/server.ts`)
- `YnabMcpServer` initializes `YNABClient`, registers tools, and handles MCP protocol
- Uses `@modelcontextprotocol/sdk` for stdio transport communication
- Tool execution returns JSON stringified results

**YNAB Client** (`src/client/YNABClient.ts`)
- Axios-based HTTP client with automatic retry (exponential backoff with jitter)
- Token bucket rate limiting (200 requests/hour YNAB limit)
- Comprehensive error transformation via `ErrorHandler`

**Tool System** (`src/tools/`)
- All tools extend `YnabTool` base class (`src/tools/base.ts`)
- Tools validate input with Zod schemas
- Registration happens in `src/tools/index.ts` via `registerTools()`

### Tool Categories
```
src/tools/
├── budgets/       # ynab_list_budgets
├── accounts/      # ynab_get_accounts
├── transactions/  # CRUD, splits, batch updates
├── categories/    # Category management and budgeting
├── payees/        # Payee CRUD
├── scheduled/     # Recurring transactions
├── transfers/     # Link/unlink transfers
├── months/        # Monthly budget data
├── imports/       # Bulk transaction import
└── analysis/      # Spending analysis and allocation
```

### Adding a New Tool

1. Create tool file in appropriate `src/tools/<category>/` directory
2. Extend `YnabTool` base class with required properties:
   - `name`: Tool identifier (prefix with `ynab_`)
   - `description`: Tool description
   - `inputSchema`: Zod schema for validation
   - `execute()`: Implementation method
3. Add to `src/tools/index.ts` imports and `registerTools()` array

Example pattern:
```typescript
import { z } from 'zod';
import { YnabTool } from '../base.js';

const InputSchema = z.object({
  budget_id: z.string(),
});

export class MyTool extends YnabTool {
  name = 'ynab_my_tool';
  description = 'What this tool does';
  inputSchema = InputSchema;

  async execute(args: unknown) {
    const input = this.validateArgs<z.infer<typeof InputSchema>>(args);
    // Use this.client for YNAB API calls
    return result;
  }
}
```

## YNAB API Notes

- **Rate Limit**: 200 requests/hour (handled automatically by RateLimiter)
- **Amounts**: Always in milliunits (multiply dollars by 1000)
- **Dates**: YYYY-MM-DD format
- **Delta Sync**: Use `last_knowledge_of_server` for efficient updates
- API token from: https://app.youneedabudget.com/settings/developer

## Configuration

Environment variables:
- `YNAB_API_TOKEN` (required) - Your YNAB API token
- `YNAB_BASE_URL` - API base URL (default: https://api.youneedabudget.com/v1)

## Versioning

Follow [semver](https://semver.org/) in `package.json`:
- **Patch** (1.0.x): Bug fixes, docs-only changes
- **Minor** (1.x.0): New tools, new parameters on existing tools (non-breaking)
- **Major** (x.0.0): Renamed/removed tools, changed return shapes, breaking parameter changes

Bump the version in the same commit as the feature/fix. Include the version in the commit message (e.g., `bump to v1.1.0`).

## Development Workflow — Red/Green TDD

All new features and bug fixes must follow red/green TDD:

1. **Red**: Write failing tests first that describe the expected behavior
2. **Green**: Write the minimum implementation to make tests pass
3. **Refactor**: Clean up while keeping tests green

Run tests continuously during development:
```bash
npm run test:watch   # Watch mode — re-runs on file changes
```

## Testing

### Test Structure
```
tests/
├── client/              # Client layer tests (RateLimiter, ErrorHandler)
├── helpers/             # Test utilities
│   ├── mockClient.ts    # Mock YNABClient factory
│   ├── fixtures.ts      # Factory functions for test data
│   └── apiResponses.ts  # Error factories
├── integration/         # Live API fixture validation
│   ├── liveApi.test.ts  # Zod schema validation tests
│   └── recordFixtures.ts # Script to record API responses
├── fixtures/recorded/   # Real API response fixtures
├── server/              # Server integration tests
└── tools/               # Tool unit tests (by category)
```

### Running Tests
```bash
npm test                 # Run all tests
npm run test:watch       # Watch mode
npm run test:coverage    # Coverage report
npm run test:record      # Record live API fixtures (requires YNAB_API_TOKEN)
```

### Writing Tool Tests
Tool tests use the mock client from `tests/helpers/mockClient.ts`:
```typescript
import { createMockClient, type MockYNABClient } from '../../helpers/mockClient.js';

let client: MockYNABClient;
let tool: MyTool;

beforeEach(() => {
  client = createMockClient();
  tool = new MyTool(client as any);
});

it('does something', async () => {
  client.someMethod.mockResolvedValue({ /* mock response */ });
  const result = await tool.execute({ budget_id: 'test' });
  expect(result).toBeDefined();
});
```

## MCP Client Configuration

```json
{
  "mcpServers": {
    "ynab": {
      "command": "node",
      "args": ["/path/to/ynab-mcp-server/dist/index.js"],
      "env": {
        "YNAB_API_TOKEN": "your_token"
      }
    }
  }
}
```

---
> Source: [adamwestland/ynab-mcp-server](https://github.com/adamwestland/ynab-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
