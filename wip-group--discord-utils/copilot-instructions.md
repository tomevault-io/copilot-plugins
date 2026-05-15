## code-style

> Code style and formatting rules for the project


# Code Style Rules

## Formatting
- **Use Biome, never Prettier** - run `bun check` to format
- Auto-organize imports with Biome
- Sort TailwindCSS classes automatically

## TypeScript
- Prefer `type` over `interface` for definitions
- Use path aliases: `@/*` for `src/*` in both apps
- Leverage tRPC's type inference - avoid manual type annotations where possible
- Strict mode enabled across all packages

## Components
- Import shadcn/ui components from `@repo/ui/components/*` or local `@/components/ui`
- Follow New York variant style for shadcn/ui components
- Check existing components before creating new ones
- Use server actions and client components appropriately in App Router

---
> Source: [wip-group/discord-utils](https://github.com/wip-group/discord-utils) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
