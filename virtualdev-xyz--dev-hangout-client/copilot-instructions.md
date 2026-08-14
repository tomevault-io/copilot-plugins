## dev-hangout-client

> - `npm run dev` - Start development server

# DevHangout Development Guide

## Commands
- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm run preview` - Preview production build
- `npm run lint` - Run ESLint
- `npm run lint:fix` - Fix ESLint issues
- `npm run format` - Format code with Prettier
- `npm run format:check` - Check formatting
- `npm run test` - Run all tests
- `npm run test:watch` - Run tests in watch mode
- `npm run test:coverage` - Run tests with coverage

## Code Style
- Use TypeScript with strict typing
- Follow ESLint and Prettier configurations
- Use functional components with hooks
- Follow container/presenter pattern
- Use Redux for global state, local state for UI
- Implement proper error handling/boundaries
- Use consistent import order: React, libraries, components, hooks, utils
- Naming: PascalCase for components, camelCase for functions/variables
- CSS: Use CSS Modules for component-specific styles
- Use 8px grid system (defined in frontend-guideline)
- Keep components small, focused and well-documented
- Follow the retro aesthetic with 8-bit inspired UI components

## State Management
- Use Redux Toolkit for state management
- Organize state by domain (game, ui, auth, etc.)
- Use entity adapters for normalized state
- Implement persisted state with redux-persist
- State is automatically persisted to localStorage
- Use the usePersistence hook for manual state operations
- Non-serializable state (Maps, Sets) should be converted to objects
- State migrations handled in persistenceTransforms.ts

---
> Source: [virtualdev-xyz/dev-hangout-client](https://github.com/virtualdev-xyz/dev-hangout-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
