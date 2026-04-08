## bitcoinschema

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- `bun dev` - Start development server with Turbopack
- `bun run build` - Build for production
- `bun run test` - Run tests with Bun
- `bun run lint` - Run Biome linting
- `bun run lint:fix` - Fix linting issues automatically
- `bun run format` - Format code with Biome
- `bun run check` - Run Biome checks and apply fixes
- `bun run type-check` - Run TypeScript type checking
- `bun run clean` - Clean build artifacts (.next and out directories)

## Architecture

This is a Next.js 15 documentation site built with:
- **Fumadocs** - Documentation framework with built-in search and navigation
- **Tailwind CSS 4** - Styling with CSS configuration
- **TypeScript** - Type safety
- **Biome** - Linting and formatting (replacing ESLint/Prettier)
- **Bun** - Package manager and test runner

### Key Structure

- `content/docs/` - MDX documentation files that are automatically processed by Fumadocs
- `src/lib/source.ts` - Fumadocs source configuration that loads MDX content
- `source.config.ts` - Fumadocs configuration for MDX processing
- `src/app/docs/[[...slug]]/page.tsx` - Dynamic route handler for documentation pages
- `biome.json` - Biome configuration with 2-space indentation, single quotes, 100 char line width

### Documentation System

The site uses Fumadocs which automatically:
- Generates navigation from MDX files in `content/docs/`
- Provides search functionality
- Handles MDX rendering with syntax highlighting
- Creates responsive documentation UI

When adding new documentation:
1. Create `.mdx` files in `content/docs/`
2. Add frontmatter with title and description
3. Files are automatically included in navigation and search

### Testing

Tests use Bun with React Testing Library setup in `src/__tests__/setup-env.ts`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/b-open-io)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/b-open-io)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
