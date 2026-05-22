## svelte-shadcn-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a specialized Model Context Protocol (MCP) server for shadcn-svelte development, adapted to use Bun instead of Node.js. It provides curated knowledge, code examples, and intelligent assistance for modern shadcn-svelte development with UI components, design systems, theming, and accessibility best practices.

## Common Commands

### Development
```bash
# Start development server with watch mode
bun run dev

# Start the MCP server
bun run start

# Install dependencies
bun install

# Inspect the MCP server (for debugging)
bunx @modelcontextprotocol/inspector bun src/index.ts
```

### Building and Publishing
```bash
# Build for production (required before publishing)
bun run build

# Publish to npm
npm publish

# Test the published package locally
bunx @binsarjr/shadcn-svelte-mcp
```

### Testing
```bash
# Test the server manually
echo '{"jsonrpc": "2.0", "id": 1, "method": "tools/list"}' | bun start

# Test via bunx
echo '{"jsonrpc": "2.0", "id": 1, "method": "tools/list"}' | bunx @binsarjr/shadcn-svelte-mcp
```

## Architecture Overview

### Core Components

- **`src/index.ts`**: Main MCP server implementation with tool and prompt handlers
- **`src/ShadcnSvelteSearchDB.ts`**: SQLite-based search database with FTS5 full-text search capabilities
- **`src/utils/config.ts`**: Configuration management with XDG directory support
- **`src/utils/jsonl.ts`**: JSONL file handling utilities for loading data
- **`src/data/knowledge/`**: Curated Q&A knowledge base for shadcn-svelte concepts
- **`src/data/examples/`**: Code examples and component implementation patterns
- **`src/data/components/`**: Complete component catalog with props and variants

### MCP Server Features

The server provides 6 main tools:
1. `search_knowledge` - Search the shadcn-svelte knowledge base
2. `search_examples` - Search code patterns and component examples
3. `search_components` - Search UI components with props and variants
4. `generate_component` - Generate components with themes and accessibility
5. `audit_with_rules` - Audit code against shadcn-svelte best practices
6. `explain_concept` - Get detailed concept explanations

Plus 4 smart prompts:
1. `generate-component` - Generate modern shadcn-svelte components
2. `audit-shadcn-code` - Audit code for accessibility and design system compliance
3. `explain-concept` - Detailed concept explanations with examples
4. `find-component` - Find specific components with usage patterns

### Database Architecture

Uses SQLite with FTS5 (Full-Text Search) for advanced search capabilities:
- **Tables**: `knowledge`, `examples`, `components`, `synonyms`
- **Virtual Tables**: `knowledge_fts`, `examples_fts`, `components_fts` for full-text search
- **Triggers**: Automatic sync between main tables and FTS tables
- **Synonyms**: shadcn-svelte-specific term expansion for better search results

### Key Technologies

- **Runtime**: Bun (requires >= 1.0.0) - runs TypeScript directly without compilation
- **Database**: SQLite with FTS5 via `bun:sqlite`
- **Validation**: Zod schemas for input validation
- **MCP SDK**: `@modelcontextprotocol/sdk` for protocol implementation

### Deployment Options

**Option 1: Direct bunx usage (Recommended)**
```json
{
  "mcpServers": {
    "shadcn-svelte": {
      "command": "bunx",
      "args": ["@binsarjr/shadcn-svelte-mcp"],
      "env": {}
    }
  }
}
```

**Option 2: Local installation**
```json
{
  "mcpServers": {
    "shadcn-svelte": {
      "command": "bun",
      "args": ["/path/to/shadcn-svelte-mcp/src/index.ts"],
      "env": {}
    }
  }
}
```

## Development Patterns

### shadcn-svelte Focus
This server is specifically designed for shadcn-svelte development patterns:
- Modern UI component development with accessibility first
- Design system consistency with Tailwind CSS
- Theme-aware components supporting light/dark modes
- Component composition and variant patterns
- TypeScript integration with proper prop typing
- Responsive design and mobile-first approach
- Accessibility compliance (WCAG guidelines)

### Search Implementation
- **Query Expansion**: Automatic synonym expansion for shadcn-svelte terms
- **Highlighted Results**: Search results include highlighted matches
- **Custom Scoring**: Advanced boosting for UI component terms
- **Relevance Ranking**: FTS5-based ranking for result ordering

### Data Management
- **Transactional Inserts**: Bulk data population using SQLite transactions
- **Automatic Indexing**: FTS triggers maintain search index consistency
- **JSON Validation**: Zod schemas ensure data integrity

## File Structure Guidelines

```
src/
├── index.ts                    # Main MCP server (tool handlers, prompt handlers)
├── ShadcnSvelteSearchDB.ts    # Database layer with search functionality
├── utils/                     # Utility functions
│   ├── config.ts              # Configuration management
│   ├── jsonl.ts               # JSONL file loading utilities
│   └── convert-to-jsonl.ts    # Data conversion utilities
└── data/
    ├── knowledge/             # Q&A knowledge base (JSONL files)
    ├── examples/              # Code examples and patterns (JSONL files)
    └── components/            # Component catalog with props and variants (JSONL files)
```

## Configuration Notes

- Database is stored at `~/.config/binsarjr/shadcn-svelte-mcp/database.db` following XDG standard
- Uses Bun's native SQLite implementation (`bun:sqlite`)
- TypeScript configuration targets ES2022 with ESNext modules
- JSON imports use `with { type: "json" }` syntax for modern module resolution
- Config directory is consistent across all platforms for simplicity

## Data Format Requirements

### Knowledge Entries
```json
{
  "question": "How do I install shadcn-svelte in my SvelteKit project?",
  "answer": "To install shadcn-svelte: 1. Run `npx shadcn-svelte@latest init` 2. Configure your project...",
  "category": "installation",
  "tags": ["setup", "cli", "initialization"]
}
```

### Example Entries
```json
{
  "title": "Basic Button Usage",
  "description": "Simple button implementation with different variants",
  "component": "Button",
  "code": "<script>\n  import { Button } from '$lib/components/ui/button';\n</script>\n\n<Button variant=\"outline\">Click me</Button>",
  "category": "buttons",
  "tags": ["button", "variants", "basic"],
  "complexity": "basic",
  "dependencies": ["@/components/ui/button"]
}
```

### Component Entries
```json
{
  "name": "Button",
  "description": "Displays a button or a component that looks like a button",
  "category": "form",
  "props": "{\"variant\": {\"type\": \"'default' | 'destructive' | 'outline'\", \"default\": \"'default'\"}}",
  "usage": "<Button variant=\"outline\">Click me</Button>",
  "installation": "npx shadcn-svelte@latest add button",
  "variants": "[\"default\", \"destructive\", \"outline\"]",
  "dependencies": "[\"class-variance-authority\"]"
}
```

## Force Resync Data

The server supports force resyncing the database with fresh data from JSONL files:

```bash
# Force resync database with updated data
bun start --force

# Or when running in dev mode
bun run dev --force
```

## Environment Variables

The following environment variables can be used to customize configuration:

- `SHADCN_SVELTE_MCP_CONFIG_DIR`: Custom config directory path
- `SHADCN_SVELTE_MCP_DB_PATH`: Custom database file path
- `XDG_CONFIG_HOME`: XDG config home directory (defaults to ~/.config)

## Important Implementation Notes

- The server uses Bun's native SQLite implementation (`bun:sqlite`) instead of external SQLite libraries
- All TypeScript files are executed directly by Bun without compilation in development
- The build process copies data files to dist/ and sets executable permissions on the main script
- JSONL files in data/ directories are automatically scanned and loaded at startup
- Database schema includes FTS5 virtual tables for full-text search with automatic triggers for synchronization

# important-instruction-reminders
Do what has been asked; nothing more, nothing less.
NEVER create files unless they're absolutely necessary for achieving your goal.
ALWAYS prefer editing an existing file to creating a new one.
NEVER proactively create documentation files (*.md) or README files. Only create documentation files if explicitly requested by the User.
```

---
> Source: [binsarjr/svelte-shadcn-mcp](https://github.com/binsarjr/svelte-shadcn-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
