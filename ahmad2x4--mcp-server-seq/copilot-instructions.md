## mcp-server-seq

> - Build: `npm run build`

# MCP-SERVER-SEQ DEVELOPMENT GUIDE

## Build & Run Commands
- Build: `npm run build`
- Start: `npm run start`
- Development: `npm run dev` (watch mode)
- Test script: `npm run test-script`
- Run specific test: `npx jest src/__tests__/your-test-file.test.ts`

## Code Style Guidelines
- **Imports**: Use ESM-style imports (`import x from 'y'`) with `.js` extension for local imports
- **Types**: Prefer explicit typing; use TypeScript interfaces for complex objects
- **Error Handling**: Use try/catch with typed errors (`error as Error`)
- **Naming**: Use camelCase for variables/functions, PascalCase for classes/interfaces
- **Structure**: Group related functions together; export only necessary items
- **Environment**: Use environment variables for configuration with defaults
- **Documentation**: Document function parameters and return values with JSDoc

## Project Structure
- `src/` - TypeScript source files
- `build/` - Compiled JavaScript output
- `prompts/` - MCP prompt templates

## MCP Server Standards
- Use zod for parameter validation
- Handle errors gracefully with meaningful error messages
- Return properly formatted JSON responses

---
> Source: [ahmad2x4/mcp-server-seq](https://github.com/ahmad2x4/mcp-server-seq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->
