## mkt-1000

> Import patterns and @scayle/storefront-nuxt preference


# Import Guidelines

## Primary Import Source
- **Always prefer importing from `@scayle/storefront-nuxt`** over `@scayle/storefront-core` or `@scayle/storefront-api`
- Use `@scayle/storefront-nuxt` as the primary import source for all storefront functionality
- Avoid direct imports from internal dependencies when possible
- The `@scayle/storefront-nuxt` package re-exports everything necessary from internal packages

## Import Patterns
- Use explicit imports instead of barrel imports (`index.ts` files that re-export everything)
- Import components and utilities directly from their source files
- Use explicit imports to improve tree-shaking and bundle optimization
- Avoid wildcard imports (`import * as`) unless absolutely necessary
- Use named imports over default imports when possible

## Nuxt-Specific Imports
- Use `#vue-router` for router utilities instead of `vue-router`
- Use `#i18n` for composables instead of `@nuxtjs/i18n`
- Use `#storefront/composables` for composables instead of `@scayle/storefront-nuxt`
- Use `#app/composables/{name}` for composables instead of `#imports`
- Use `#imports` for imports within a pure server context

## File-Specific Import Guidelines
- Import from `app/composables/` for custom composables
- Import from `app/utils/` for utility functions
- Import from `app/constants/` for constant values
- Import from `modules/ui/runtime/` for base components
- Import from `rpcMethods/` for RPC method implementations

## TypeScript Import Guidelines
- Use explicit type imports when importing only types
- Use `import type` for type-only imports
- Use proper type imports for better tree-shaking
- Consider using type-only imports for better performance

## Performance Considerations
- Use explicit imports to improve tree-shaking
- Avoid importing entire modules when only specific functions are needed
- Consider using dynamic imports for code splitting
- Monitor bundle size impact of import changes
- Use appropriate import patterns for better performance

## Best Practices
- Group imports logically (external, internal, relative)
- Use consistent import ordering
- Avoid circular dependencies
- Use absolute imports when possible
- Consider using path aliases for cleaner imports

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/riadler89)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/riadler89)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
