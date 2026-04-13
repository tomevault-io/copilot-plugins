## linear-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Setup
1. Create a `.env` file based on `.env.example` and add your Linear API key:
   ```
   LINEAR_API_KEY=your_linear_api_key_here
   ```

## Build Commands
- Build: `npm run build` (compiles TypeScript and makes executable)
- TypeCheck: `tsc --noEmit`

## Code Style Guidelines
- **Module System**: ES Modules with .js extension in imports (e.g., `import { x } from './y.js'`)
- **Formatting**: TypeScript with strict type checking enabled
- **Types**: Use Zod for runtime validation/schemas (z.string(), etc.)
- **Constants**: UPPER_SNAKE_CASE for constants
- **Naming**: camelCase for variables/functions, PascalCase for classes/types
- **Imports**: Group imports by source (SDK, then utilities, then local modules)
- **Error Handling**: Return structured responses with appropriate content type
- **MCP Implementation**: Follow Model Context Protocol patterns for tools/resources
- **Comments**: Add descriptive comments for main component sections

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/peterchappy)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/peterchappy)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
