## project-context

> - Framework: `tailwindcss`, `Typescript`, `nextjs`,`Framer motion`


# General rules

## Environment
- Project: `react`
- Framework: `tailwindcss`, `Typescript`, `nextjs`,`Framer motion`
- Package Manager: `npm`
- Development Server: Run with `npm run dev`
- API State: `TanStack Query`
- Global state : `Zustand`

## General folder structure
- `api`: API endpoints and queries
- `public/`: Static assets, icons and fonts
- `src/`: Source code
  - `Page/`: Individual concepts
    - Each page has its own directory with components, hooks, and utilities
  - `components/`: Shared React components
    - `layout/`: Layout components
    - `ui/`: UI components
  - `config/`: Configuration files 
  - `contexts/`: React context providers
  - `hooks/`: Custom React hooks
  - `providers/`: React providers
  - `lib/`: Libraries and utilities
  - `stores/`: State management
  - `types/`: TypeScript type definitions
  - `utils/`: Utility functions 
  - `global.css/`: All the global level styles

### App Structure
-App components follow the naming convention `[AppName] AppComponent.tsx`
- Page are organized in `src/apps/[page-name]/` directories
- Each page folder typically contains:
  - `components/`: App-specific components
  - `hooks/`: Custom hooks specific to the app
  - `utils/`: Utility functions for the app
  - `types/`: App-specific type definitions
- The main app component is exported from `src/apps/[app-name]/index.tsx`
- Page components receive common props via the `PageProps` interface

### State Management
- Page use a combination of local state (React hooks) and global state (stores)
- Stores are defined in `src/stores/` with a naming pattern of `use[Store]Store`
- Stores are implemented using a state management library (Zustand)

### Component Organization
- Shared UI components from `@/components/ui/`
- Custom hooks from `@/hooks/` provide reusable functionality
- Apps leverage shared components but define app-specific UI in their own components folder

---
> Source: [vignesh33654/traintracking](https://github.com/vignesh33654/traintracking) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
