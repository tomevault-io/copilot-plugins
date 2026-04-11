## poke-start

> - `pnpm dev` - Start development server

# Agent Guidelines

## Build/Lint/Test Commands

- `pnpm dev` - Start development server
- `pnpm build` - Build for production
- `pnpm start` - Start production server
- `pnpm format` - Format code with Prettier
- No test framework configured in this project

## Code Style

- **Framework**: SolidJS with TypeScript
- **Imports**: Use `~/` for src imports, external imports first
- **Formatting**: No semicolons, single quotes (Prettier configured)
- **Components**: Interface props, function components, PascalCase naming
- **JSX**: Use `class` not `className`, preserve JSX syntax
- **Types**: Strict TypeScript, explicit interfaces for props
- **Files**: `.tsx` for components, `.ts` for utilities
- **Naming**: PascalCase components, camelCase variables/functions
- **CSS**: Tailwind classes, utility-first approach
- **Database**: Use Neon serverless with `sql` template literals
- **Routing**: SolidJS Router with route definitions and createAsync

## Environment

- Node >=20, pnpm >=9 required
- Database URL in .env file
- Module type: ESM

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/atilafassina)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/atilafassina)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
