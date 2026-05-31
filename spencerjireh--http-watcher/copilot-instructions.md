## http-watcher

> - `npm run dev` - Start development server with Electron

# HTTP Watcher - Agent Guidelines

## Build Commands

- `npm run dev` - Start development server with Electron
- `npm run build` - Build for production
- `npm run build-main` - Build main Electron process only
- `npm start` - Start Electron app in development mode

## Lint & Format Commands

- `npm run lint` - Check code for ESLint issues
- `npm run lint:fix` - Auto-fix ESLint issues
- `npm run format` - Format code with Prettier
- `npm run format:check` - Check code formatting

## Code Style Guidelines

### TypeScript & Imports

- Strict TypeScript enabled, target ES2022
- Import React: `import React from 'react'`
- Use type imports: `import type { HttpRequest } from '@shared/types'`
- Path aliases: `@/*` for renderer, `@shared/*` for shared types

### Component Conventions

- Functional components with explicit return types or React.FC
- Props interfaces in shared/types.ts
- useState with explicit types: `useState<HttpRequest[]>([])`
- Async functions with try/catch error handling

### Styling

- Tailwind CSS only, no inline styles
- Custom color scheme: bg-bg-dark, text-text-primary, etc.

### Error Handling

- try/catch for async operations
- console.error for logging
- IpcResult objects for IPC with success/error fields

### File Structure

- src/main/ - Electron main process
- src/renderer/ - React app
- src/shared/types.ts - Shared types
- src/preload/ - Preload scripts

### Naming

- Components: PascalCase (RequestList.tsx)
- Functions/variables: camelCase
- Constants: UPPER_SNAKE_CASE

### General Rules

- Do not use emojis in comments or code

---
> Source: [spencerjireh/http-watcher](https://github.com/spencerjireh/http-watcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
