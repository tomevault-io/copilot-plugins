## typescript-standards-rules

> TypeScript formatting, linting rules, and best practices.

# TypeScript Standards
# Last Updated: 2025-03-31 10:13:02 AM

## Formatting (Prettier)
- maxLineLength: 100
- indentSize: 2
- indentStyle: space
- formatter: prettier

## Linting Rules
- noUnusedVars: error
- noUnescapedEntities: error
- noExplicitAny: warn
- noImgElement: warn
- exhaustiveDeps: warn

## Best Practices

### HTML Entities
- Use "&apos;" for apostrophes
- Use "&quot;" for quotes

### Imports
- Prefer destructuring
- Organize imports on save
- Remove unused imports

### Types
- Avoid using "any" types
- Prefer explicit types
- Use built-in utility types when possible

### Hooks
- Include all dependencies in dependency arrays
- Use useCallback for event handlers
- Use useMemo for expensive computations 

---
> Source: [drumnation/unsplash-smart-mcp-server](https://github.com/drumnation/unsplash-smart-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
