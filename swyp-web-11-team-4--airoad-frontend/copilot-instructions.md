## airoad-frontend

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a React + TypeScript + Vite frontend application for Team 4's SWYP Web project (11기). The project uses:
- **Vite** with SWC for fast builds and hot module replacement
- **React 19** with TypeScript in strict mode
- **Biome** for linting and formatting (instead of ESLint/Prettier)
- **pnpm** as the package manager

## Commands

### Development
```bash
pnpm dev          # Start development server with HMR
pnpm build        # Type check and build for production
pnpm preview      # Preview production build locally
```

### Code Quality
```bash
pnpm lint         # Check code with Biome linter
pnpm lint:fix     # Auto-fix linting issues
pnpm format       # Check code formatting
pnpm format:fix   # Auto-format code
```

## Architecture

### Project Structure (Feature-Sliced Design)
This project follows Feature-Sliced Design principles with a clear separation of concerns:

#### Layer Structure

**Pages** (`pages/[page-name]/`):
- Page components organized in `ui/` subdirectory
- Structure: `pages/[page-name]/ui/index.tsx` + `index.css.ts`
- Example: `pages/users/ui/index.tsx` with `index.css.ts`
- Pages coordinate entities and features, contain no business logic
- Export page component from `pages/[page-name]/index.ts`

**Entities** (`entities/[entity]/`):
- **API**: Contains GET requests and data fetching logic (queries)
  - Example: `entities/user/api/user.queries.ts` handles user data fetching
  - Query hooks using React Query/TanStack Query for data fetching
  - File naming: `[entity].queries.ts`
- **UI**: Entity-specific UI components in dedicated folders
  - Example: `entities/user/ui/user-list/user-list.tsx`
  - Folder: `hyphen-case`, File: `hyphen-case`
- **Mocks**: GET request mock handlers and fixtures
  - `mocks/[entity].fixtures.ts` - Mock data
  - `mocks/[entity].handlers.ts` - GET request handlers
  - Example: `entities/user/mocks/user.handlers.ts`

**Features** (`features/[action-verb]/`):
- Contains POST, PUT, DELETE, PATCH and other mutation logic
- Feature folders use verb-based naming with `hyphen-case` (e.g., `create-user`, `update-profile`, `delete-post`)
- **API**: Mutation hooks and business logic for state changes
  - Example: `features/create-user/api/create-user.mutation.ts`
  - File naming: `[feature].mutation.ts`
- **Mocks**: POST, PUT, DELETE, PATCH mutation mock handlers
  - `mocks/[feature].handlers.ts` - Mutation handlers
  - Example: `features/create-user/mocks/create-user.handlers.ts`

**App** (`app/`):
- **Providers**: All application-level providers
  - `providers/query.tsx` - React Query provider
  - `providers/router.tsx` - React Router setup
  - `providers/index.tsx` - Combined providers
- **Mocks**: MSW configuration
  - `mocks/browser.ts` - Worker setup with all handlers
  - `mocks/init.ts` - Mock initialization logic
  - All entity and feature handlers are registered here

**Shared** (`shared/`):
- **Lib**: Shared utilities and configurations
  - `lib/axios.ts` - Axios client instance
  - `lib/msw.ts` - MSW factory functions (`createHandler`, `createHandlers`)
- **UI**: Reusable UI components (buttons, inputs, etc.)
- **Types**: Shared TypeScript types and interfaces

**Key Principles**:
- Folder naming: Always use `hyphen-case` (e.g., `create-user`, `user-list`)
- GET requests → `entities/[entity]/api/` and `entities/[entity]/mocks/`
- Mutations (POST, PUT, DELETE, PATCH) → `features/[action-verb]/api/` and `features/[action-verb]/mocks/`
- All providers managed in `app/providers/`
- MSW handlers registered in `app/mocks/browser.ts`

### Build Configuration
- **Vite config**: Uses `@vitejs/plugin-react-swc` for Fast Refresh via SWC (not Babel)
- **TypeScript**: Project references split between app code (`tsconfig.app.json`) and build tooling (`tsconfig.node.json`)
- **Strict mode**: TypeScript strict mode enabled with additional checks for unused locals/parameters

### UI Component Structure
All UI components must follow this structure using **Vanilla Extract** for styling:

```
component-name/
├── index.tsx      # Component logic
└── index.css.ts   # Vanilla Extract styles
```

**Styling Guidelines**:
- **NEVER** use inline styles (`style={{ ... }}`)
- **ALWAYS** use Vanilla Extract (`.css.ts` files) for all styling
- Use descriptive class names exported as named exports
- Folder names use `hyphen-case` (e.g., `user-list/`)
- Component files are always `index.tsx` and `index.css.ts`

**Example**:
```typescript
// user-list/index.css.ts
import { style } from "@vanilla-extract/css";

export const container = style({
  padding: "20px",
});

export const title = style({
  fontSize: "24px",
  fontWeight: "600",
});

// user-list/index.tsx
import * as styles from "./index.css";

export const UserList = () => {
  return (
    <div className={styles.container}>
      <h2 className={styles.title}>Title</h2>
    </div>
  );
};
```

### Code Style

#### Linting & Formatting
- **Biome** handles both linting and formatting
- Line width: 100 characters
- Indent: 2 spaces
- Key rules: `noExplicitAny`, `useExhaustiveDependencies`, `noNonNullAssertion` set to warn
- Organize imports automatically enabled

#### Naming Conventions
- **Folders**: `hyphen-case` (e.g., `create-user`, `user-list`)
- **Files**: `hyphen-case` (e.g., `user-list.tsx`, `user.queries.ts`)
- **Components**: `PascalCase` in export (e.g., `export const UserList`)
- **Variables & Functions**: `camelCase` (e.g., `fetchUsers`, `userName`)
- **Types & Interfaces**: `PascalCase` (e.g., `User`, `QueryProviderProps`)
- **Constants**: `UPPER_SNAKE_CASE` (e.g., `API_BASE_URL`)

#### Function Declaration
- **All functions**: Use Arrow Functions
  ```typescript
  // General functions
  const fetchUsers = async (): Promise<User[]> => {
    const { data } = await api.get<User[]>("/users");
    return data;
  };

  // Components
  export const UserList = () => {
    return <div>...</div>;
  };

  // Custom Hooks
  export const useUsers = () => {
    return useSuspenseQuery(userQueries.list());
  };

  // Event Handlers
  const handleSubmit = (event: FormEvent) => {
    event.preventDefault();
  };
  ```

#### File Naming with Role
- API files should include role suffix:
  - Queries: `[entity].queries.ts` (e.g., `user.queries.ts`)
  - Mutations: `[feature].mutation.ts` (e.g., `create-user.mutation.ts`)
- Mock files should include role suffix:
  - Handlers: `[entity].handlers.ts` or `[feature].handlers.ts`
  - Fixtures: `[entity].fixtures.ts`

#### Type Definitions
- Use `interface` for object shapes
- Use `type` for unions, intersections, and primitives
  ```typescript
  interface User {
    id: number;
    name: string;
  }

  type UserRole = "admin" | "user" | "guest";
  ```

### Entry Points
- **HTML**: `index.html` (root level, Vite convention)
- **App entry**: `src/main.tsx` renders `App.tsx` in React StrictMode
- **Root element**: `#root` in index.html

## Notes
- The React Compiler is not compatible with SWC (see project README)
- Use `pnpm` for all package management operations
- Biome lints only `src/**/*.ts` and `src/**/*.tsx` files

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/swyp-web-11-team-4)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/swyp-web-11-team-4)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
