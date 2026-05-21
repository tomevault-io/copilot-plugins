## typescript

> Rules for TypeScript files


# TypeScript Guidelines

- Files should end with a new line
- Use ES modules (`import`/`export`)
- Use `type` for type imports
- When importing Node.js modules, don't use specifiers for `fs` and `path`
- Use `pathe` instead of `path` module for Windows compatibility
- Formatting:
  - Prettier with: `printWidth: 80, singleQuote: true, trailingComma: all`
  - Use 2 spaces for indentation
- Use TypeScript's strict mode
- Use `zod` for runtime type validation
- For MCP tools, follow the [MCP spec](mdc:https:/github.com/ModelContractProtocol/spec) and use `fastmcp` for implementation
- Use the `effect` library for functional programming patterns when appropriate

---
> Source: [umijs/umi-mcp](https://github.com/umijs/umi-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
