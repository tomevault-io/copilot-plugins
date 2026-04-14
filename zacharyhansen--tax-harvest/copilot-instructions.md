## tax-harvest

> This is a monorepo using Turborepo with pnpm workspaces:

# Tax Harvest - Global Project Rules

## Project Structure
This is a monorepo using Turborepo with pnpm workspaces:
- `apps/api` - NestJS GraphQL API with Prisma
- `apps/web` - Next.js frontend with Apollo Client  
- `packages/ui` - Shared UI components
- `packages/typescript-config` - Shared TypeScript configs

## Development Commands
Use these Turbo commands for development:
```bash
pnpm dev          # Start development servers
pnpm build        # Build all packages
pnpm lint         # Lint all packages
pnpm check:types  # Type check all packages
pnpm test         # Run tests
pnpm test:coverage # Run tests with coverage
```

## Code Standards
- Use TypeScript for all code
- Follow existing code patterns and conventions
- Add JSDoc descriptions and examples when modifying functions/components
- Use Biome for linting and formatting
- Use conventional commits for commit messages

## UI/Styling Guidelines
- Import UI components from `@repo/ui/...`
- Use only Tailwind CSS for styling
- Use colors from `@packages/ui/src/shadcn-theme.css`
- Follow existing component patterns

## GraphQL
- Schema changes auto-update when files are saved
- Use generated types from codegen

## Testing
- Use Vitest for testing
- Maintain test coverage
- Follow existing test patterns

## File Conventions
- Use kebab-case for file names
- Group related files in directories
- Export from index files where appropriate

## Security
- Never expose or log secrets/keys
- Follow security best practices
- Use proper authentication patterns

## Performance
- Optimize bundle sizes
- Use proper caching strategies
- Follow React performance best practices

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zacharyhansen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
