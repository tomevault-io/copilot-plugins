## 3diso

> - `terminal-setup.cmd` - Configure VSCode terminal settings

# 3D Isometric Project Guidelines

## Setup & Configuration
- `terminal-setup.cmd` - Configure VSCode terminal settings
  - Sets up Shift+Enter key binding for terminal interaction
  - Configures proper terminal profiles for Windows

## Build & Dev Commands
- `npm run dev` - Run development server with Turbopack
- `npm run build` - Build for production
- `npm run start` - Start production server
- `npm run lint` - Run ESLint

## Code Style Guidelines
- **TypeScript**: Use strict type checking with explicit typing
- **Imports**: Group imports by source (React, Next.js, components, etc.)
- **Components**: Use functional components with TypeScript interfaces
- **Naming**: PascalCase for components, camelCase for functions/variables
- **CSS**: Use Tailwind utility classes with consistent ordering
- **Error Handling**: Use try/catch blocks with typed error objects
- **File Structure**: Follow Next.js 14+ app router conventions
- **State Management**: Use React hooks (useState, useReducer) for component state
- **Formatting**: 2-space indentation, semicolons, trailing commas
- **Accessibility**: Ensure proper ARIA attributes and semantic HTML

## Folder Structure
- Use `src/` directory with app router architecture
- Place reusable components in `src/components/`
- Follow path aliases with `@/` prefix as configured in tsconfig

---
> Source: [spotcircuit/3diso](https://github.com/spotcircuit/3diso) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
