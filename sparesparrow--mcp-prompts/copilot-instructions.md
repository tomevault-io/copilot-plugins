## root

> - Use pnpm for all package operations

# MCP Prompts Monorepo Root Rules

## Build & Package Management
- Use pnpm for all package operations
- Build order: core → adapters → apps/server
- Use `pnpm -r build` for full monorepo builds
- Test with `pnpm -r test`

## Import Strategy
- Use workspace:* for internal dependencies
- Import from built outputs: @mcp-prompts/core/dist/
- Avoid circular dependencies

## Code Style
- ESLint 9.0+ flat config
- Prettier for formatting
- TypeScript strict mode
- Use NodeNext module resolution

## Architecture
- Hexagonal architecture patterns
- Ports (interfaces) in core/ports/
- Adapters in separate packages
- Pure domain logic in core/ # MCP Prompts Monorepo Root Rules

---
> Source: [sparesparrow/mcp-prompts](https://github.com/sparesparrow/mcp-prompts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
