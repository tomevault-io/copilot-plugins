## twitter-mcp-server

> - src/                -> MCP server source code.

# Cursor Rules

# Project Structure Summary (Updated)

## Root Files
- package.json
- package-lock.json
- tsconfig.json

## Key Directories
- src/                -> MCP server source code.
- agent-twitter-client/ -> Twitter agent client project containing its own src/, test-assets/, configuration files (tsconfig.json, rollup.config.mjs, package.json, README.md, etc.).

## Dependencies
- Multiple npm modules are present as specified in package-lock.json (e.g., querystringify, raw-body, twitter-api-v2, etc.).

## Guidelines
- Keep the code clean, modular, efficient, and readable.
- Follow the checklist for integration and development process.
- When adding new tools or making modifications, ensure tests are updated and documentation is maintained.
- Don't forget to build the project after making changes to the typescript files!

---
> Source: [taazkareem/twitter-mcp-server](https://github.com/taazkareem/twitter-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
