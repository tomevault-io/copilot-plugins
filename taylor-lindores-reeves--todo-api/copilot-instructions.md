## todo-api

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands
- Build: `bun run build`
- Start development server: `bun run dev`
- Production start: `bun start`
- Lint: `bun run lint`
- Generate API: `bun run generate`

## Code Style Guidelines
- **Imports**: Use alias paths from tsconfig (e.g., `@todo/*`) and organize imports with Biome
- **Formatting**: Follow Biome.js rules (enabled in biome.json)
- **Types**: Use TypeScript with strict mode; use Zod for API schema validation
- **Naming**: Use kebab-case for files, camelCase for variables/functions, PascalCase for components
- **Error Handling**: Use appropriate HTTP status codes with TypedNextResponse
- **API Structure**: Use next-rest-framework for route definition and validation
- **Components**: Use client components with "use client" directive when needed
- **CSS**: Use Tailwind CSS with class merging utility (cn function)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/taylor-lindores-reeves)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/taylor-lindores-reeves)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
