## opencode-plugin-simple-memory

> - **Install**: `bun install`

# Agent Guidelines

## Commands
- **Install**: `bun install`
- **Type check**: `bun run tsc --noEmit`
- **Run**: `bun run index.ts`
- **Test manually**: `bun -e "import { MemoryPlugin } from './index.ts'; ..."`

## Code Style
- **Runtime**: Bun (use Bun APIs: `Bun.file()`, `Bun.write()`, `Bun.Glob`, `Bun.$`)
- **Imports**: Use `import type` for type-only imports (`verbatimModuleSyntax`)
- **Types**: Strict mode enabled, handle `undefined` from indexed access (`noUncheckedIndexedAccess`)
- **Naming**: camelCase for functions/variables, PascalCase for types/interfaces
- **Exports**: Re-export public API from `index.ts`, implementation in `src/`

## Plugin Structure
- Tools use `@opencode-ai/plugin` `tool()` helper with Zod-like schema (`tool.schema`)
- Plugin exports async function returning `{ tool: { ... } }`
- Memories stored in `.opencode/memory/` as logfmt files

---
> Source: [cnicolov/opencode-plugin-simple-memory](https://github.com/cnicolov/opencode-plugin-simple-memory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
