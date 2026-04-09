## multiforum-backend

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build/Run Commands
- `npm run codegen` - Generate GraphQL code
- `npm run tsc` - Run TypeScript compiler
- `npm run build` - Build project (runs tsc + copies Cypher files)
- `npm run start` - Start the server
- `npm run logSchema` - Log GraphQL schema to console
- No explicit test/lint commands defined (update when added)

## Code Style
- **Imports**: ES modules with .js extensions, grouped (third-party first, then local)
- **Naming**: camelCase for variables/functions, PascalCase for types/classes, UPPER_SNAKE for constants
- **Types**: Always use explicit TypeScript types with strict mode
- **File Structure**: Domain-based organization (rules/, customResolvers/) with feature-based subfolders
- **Error Handling**: Early returns, explicit error messages, proper error propagation
- **GraphQL**: Separate Cypher queries in .cypher files, custom resolvers for complex operations
- **Conventions**: 
  - ES2018 target with ESNext modules
  - Use Neo4j GraphQL library and OGM for database operations
  - Follow existing permission system architecture (see README.md)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gennit-project)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gennit-project)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
