## deep-code-reasoning-mcp

> - `npm run build` - Build TypeScript to dist/

# Agent Guidelines for Deep Code Reasoning MCP

## Build/Test/Lint Commands
- `npm run build` - Build TypeScript to dist/
- `npm run dev` - Watch mode development
- `npm test` - Run Jest tests with ES module support
- `npm run lint` - ESLint with TypeScript rules
- `npm run typecheck` - Type checking without emit
- Single test: `npm test -- --testNamePattern="specific test name"`

## Architecture
- **MCP Server**: Bridges Claude Code with Gemini 2.5 Pro for complex code analysis
- **Core**: src/index.ts - Main MCP server with 10 analysis tools
- **Analyzers**: src/analyzers/DeepCodeReasonerV2.js - Gemini integration
- **Models**: src/models/types.ts - TypeScript interfaces for all data structures
- **Tools**: Conversational AI analysis, hypothesis tournaments, execution tracing
- **Services**: Multi-model workflow routing (Claude→Gemini escalation)

## Code Style
- **Imports**: ES modules (.js extensions required), no default imports mixing
- **Types**: Strict TypeScript, Zod validation for API parameters
- **Naming**: snake_case for API parameters, camelCase for internal code
- **Format**: 2-space indent, single quotes, trailing commas, semicolons
- **Errors**: ErrorClassifier for consistent error handling, McpError for API responses
- **Validation**: InputValidator for all external inputs, comprehensive file path validation

---
> Source: [evalops/deep-code-reasoning-mcp](https://github.com/evalops/deep-code-reasoning-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
