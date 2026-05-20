## mcp-testrail

> This is an MCP (Model Context Protocol) server for TestRail integration built with TypeScript. When working on this project, always reference the documentation in the `AI/` directory for context and guidance.

# Claude Instructions for MCP TestRail Project

## Project Overview
This is an MCP (Model Context Protocol) server for TestRail integration built with TypeScript. When working on this project, always reference the documentation in the `AI/` directory for context and guidance.

## Development Workflow
Before making any changes, review `AI/development-workflow.md` for the complete development process including:
- Environment setup
- Build and test commands
- Debugging procedures
- Project structure

## Key Commands to Remember
- **Development**: `npm run start:fastmcp`
- **Build**: `npm run build`
- **Test**: `npm test`
- **Format**: `npm run format`
- **Debug**: `npm run debug`
- **Production**: `npm run start:stdio`

## Project Structure
```
src/
├── client/       # TestRail API client implementation
├── server/       # MCP server implementation
├── shared/       # Common schemas and types
├── stdio.ts      # stdio mode entry point
└── sse.ts        # SSE mode entry point
```

## Important Notes
- Always run tests after making changes: `npm test`
- Use Biome for code formatting: `npm run format`
- Node.js version requirement: 20.18.1 ~ 22.14.0
- The project uses ES modules (`"type": "module"`)
- MCP Inspector is available for debugging via `npm run debug`

## When Making Changes
1. Check existing documentation in `AI/` directory
2. Follow the development workflow in `AI/development-workflow.md`
3. Run tests and formatting before committing
4. Update relevant documentation if adding new features

## Environment Variables Required
- `TESTRAIL_URL`: Your TestRail instance URL
- `TESTRAIL_USERNAME`: TestRail username
- `TESTRAIL_API_KEY`: TestRail API key

---
> Source: [bun913/mcp-testrail](https://github.com/bun913/mcp-testrail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
