## langfuse-prompt-mcp

> Guidance for Claude Code when working with this TypeScript MCP server.

# CLAUDE.md

Guidance for Claude Code when working with this TypeScript MCP server.

## Project Structure

```
src/                    # TypeScript source
├── orchestrator/       # Queen Bee parallel processing
├── handlers/           # Tool request handlers
├── evaluators/         # Scoring logic
├── improvers/          # Enhancement techniques
└── types/              # TypeScript definitions

dist/                   # Compiled JavaScript (auto-generated)
```

## Key Commands

```bash
npm run build          # Compile TypeScript → JavaScript
npm start              # Run server (dist/server.js)
npm test               # Run tests
```

## Tool Integration

Each MCP tool follows this pattern:
1. Schema defined in `src/tools/definitions.ts`
2. Handler in `src/handlers/{tool}.ts`
3. Returns MCP-formatted responses

## Special Patterns

### LLM Evaluation
Returns `action: 'require_claude_task'` to delegate to Claude Code's Task tool.

### Environment
Reads from `$HOME/.claude/.env` for Langfuse credentials.

### Known Issues
- Circular dependency avoided in config.ts using console logging
- TypeScript strict mode temporarily relaxed
- Some type errors remain (non-blocking)

## Adding Features

1. Create TypeScript files in `src/`
2. Run `npm run build` to compile
3. Test with `npm start`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dlewiski) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
