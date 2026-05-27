## mcp-client-app

> - `pnpm dev`: Start the development server

# MCP Client App Development Guidelines

## Build & Development Commands
- `pnpm dev`: Start the development server
- `pnpm build`: Run typecheck and build the app
- `pnpm build:mac`: Build for macOS
- `pnpm build:win`: Build for Windows
- `pnpm build:linux`: Build for Linux

## Lint & Format Commands
- `pnpm lint`: Run ESLint
- `pnpm format`: Format code with Prettier
- `pnpm typecheck`: Run TypeScript type checking

## Code Style Guidelines
- **TypeScript**: Use strict typing with proper interfaces/types
- **React**: Use functional components with hooks
- **Imports**: Group imports (React, libraries, local) with blank line between groups
- **Naming**: PascalCase for components, camelCase for variables/functions
- **Error Handling**: Use try/catch for async operations
- **Project Structure**:
  - `main`: Electron main process (IPC handlers, services)
  - `preload`: Electron preload scripts
  - `renderer`: React UI components
- **State Management**: Use React Context API for shared state

---
> Source: [RegiByte/mcp-client-app](https://github.com/RegiByte/mcp-client-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
