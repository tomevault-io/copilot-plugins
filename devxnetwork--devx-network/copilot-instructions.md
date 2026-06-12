## devx-network

> - **Dev server**: `bun run dev` (starts Next.js dev server on port 3000)

# Agent Guidelines for DEVx.network

## Build/Lint/Test Commands

- **Dev server**: `bun run dev` (starts Next.js dev server on port 3000)
- **Build**: `bun run build` (creates production build)
- **Lint**: `bun run lint` (runs ESLint with Next.js rules)
- **Format**: `bun run format` (auto-formats with Prettier)
- **Type check**: `bunx tsc --noEmit` (no test framework configured)

## Code Style Guidelines

- **Formatting**: Use tabs, no semicolons, double quotes, 100 char line width
- **React**: Use "use client" directive for client components, functional components only
- **Imports**: Prefer named imports, use @ alias for project root imports
- **TypeScript**: Strict mode enabled, use explicit types for props and state
- **Styling**: Use styled-components exclusively (NO Tailwind - refactoring in progress)
- **State**: Use React hooks (useState, useEffect), avoid class components
- **Async**: Use async/await over promises, handle errors with try/catch
- **File naming**: PascalCase for components (Footer.tsx), camelCase for utilities
- **Exports**: Named exports for components, default for pages
- **Database**: Use better-sqlite3 with prepared statements for SQLite operations
- **Next.js**: Follow App Router conventions, use server/client components appropriately

## Documentation Index

### Project Setup

- **[README.md](./README.md)** - When to read: Initial project setup and contribution guidelines. Summary: Development setup, installation, and contribution workflow.

### Code Conventions

- **[docs/conventions/file-conventions.md](./docs/conventions/file-conventions.md)** - When to read: Before creating or refactoring any code files. Summary: Four-section file structure (types, constants, components, functions) with exports-first ordering, clean comment headers, and utils separation.
- **[docs/conventions/styling-guidelines.md](./docs/conventions/styling-guidelines.md)** - When to read: When working with styles or migrating from Tailwind. Summary: Styled-components usage, naming conventions, and Tailwind migration guide.

---
> Source: [DEVxNetwork/devx.network](https://github.com/DEVxNetwork/devx.network) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-12 -->
