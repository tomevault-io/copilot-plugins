## next-portfolio

> - `npm run dev` - Start development server (with Turbopack)

# CLAUDE.md - Instructions for AI Assistants

## Commands
- `npm run dev` - Start development server (with Turbopack)
- `npm run build` - Build for production
- `npm run start` - Start production server
- `npm run lint` - Run Next.js linting

## Code Style Guidelines
- **TypeScript**: Use strict typing; explicit return types for components
- **Components**: Functional components with explicit types; PascalCase naming
- **Imports**: Order - external libs, types (using `import type`), internal modules
- **Path Aliases**: Use `@/*` for imports from root directory
- **Naming**: PascalCase for components, camelCase for functions/variables
- **CSS**: Use Tailwind for styling
- **Formatting**: Maintain consistent indentation (2 spaces)
- **JSX**: Keep components focused, extract complex logic to hooks
- **Error Handling**: Use try/catch for async operations
- **Props**: Use Readonly<{}> for component props
- **State Management**: Prefer useState for local state
- **Structure**: Follow Next.js App Router pattern (page.tsx, layout.tsx)
- **Exports**: Use default exports for components, named for utilities
- **Comments**: Minimal, focused on "why" not "what"

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rsypchenko) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
