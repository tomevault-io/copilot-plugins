## restaurant-passport-app

> - `bun run dev` - Start development server

# Restaurant Passport App - Developer Guide

## Commands

- `bun run dev` - Start development server
- `bun run build` - Build for production
- `bun run lint` - Run ESLint
- `bun run storybook` - Start Storybook (UI component explorer)
- `bun run new` - Generate new components using Plop
- `bun run drizzle:generate` - Generate database migrations
- `bun run drizzle:migrate` - Run database migrations
- `bun run scrape` - Run restaurant data scraper
- `bun run test` - Run tests with Vitest (to run single test: `bun run test path/to/test.spec.ts`)

## Code Style Guidelines

- **TypeScript**: Use strict mode, explicit return types on functions
- **Imports**: Group imports (React, third-party, internal) with blank line separators
- **Formatting**: 2-space indentation, avoid console.log in production code
- **Components**: Use functional components with React.forwardRef when needed
- **Naming**: PascalCase for components, camelCase for functions, kebab-case for files
- **Error Handling**: Use try/catch with specific error types, avoid generic Error types
- **State Management**: Use React hooks (useState, useContext) for state
- **Database**: Use Drizzle ORM for all database operations
- **API Routes**: Create Next.js API routes with proper status codes and error handling
- **Mobile**: Ensure responsive design with mobile-first approach
- **Testing**: Write tests with Vitest, follow component testing patterns
- **UI Components**: Use Radix UI primitives with Tailwind for styling
- **Performance**: Optimize component rendering, minimize rerenders[byterover-mcp]

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Monichre) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
