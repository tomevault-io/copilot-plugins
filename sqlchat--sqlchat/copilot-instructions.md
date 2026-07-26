## sqlchat

> - Development: `pnpm dev` (or `pnpm nodb` without DB, `pnpm usedb` with DB)

# SQL Chat - Development Guide

## Build/Run Commands
- Development: `pnpm dev` (or `pnpm nodb` without DB, `pnpm usedb` with DB)
- Build: `pnpm build`
- Production: `pnpm start`
- Lint: `pnpm lint`
- Database: `pnpm prisma migrate dev` (setup), `pnpm prisma db seed` (seed)

## Code Style Guidelines
- **Formatting**: 140 char width, double quotes, semicolons (Prettier)
- **TypeScript**: Strict types, interfaces for structure, generics when appropriate
- **Components**: Functional components with hooks, PascalCase naming
- **Imports**: External libs first, followed by internal modules with @/ alias
- **Naming**: PascalCase for components/interfaces, camelCase for variables/functions
- **Functions**: Descriptive verb-noun naming (handleClick, fetchData)
- **Error Handling**: try/catch blocks with toast notifications for user-facing errors
- **File Structure**: Feature-based organization in /components, /types, /store

## State Management
- Zustand for global state
- React hooks for component state
- TypeScript types for state shape

---
> Source: [sqlchat/sqlchat](https://github.com/sqlchat/sqlchat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
