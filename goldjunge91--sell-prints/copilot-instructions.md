## sell-prints

> This document provides guidance for the Gemini code assistant to effectively contribute to this project.

# Gemini Code Assistant Guidance

This document provides guidance for the Gemini code assistant to effectively contribute to this project.

## Project Overview

This is a 3D printing marketplace built with Next.js, TypeScript, and Tailwind CSS. The primary goal is to allow users to upload 3D models, get instant price quotes, and place orders for 3D prints.

## Tech Stack

- **Framework**: Next.js 15 (App Router)
- **Language**: TypeScript (strict mode)
- **Styling**: Tailwind CSS with shadcn/ui components
- **3D Rendering**: Three.js / React Three Fiber
- **Database ORM**: Drizzle
- **Authentication**: NextAuth.js
- **Package Manager**: pnpm

## !! most important rule 

A component should do one thing well, not ten things poorly.
```react
// Bad
// DONT DO THIS BAD PATTERN 
function UserProfile() {
  const [data, setData] = useState(null);
  useEffect(() => {
    fetchUser().then(setData);
  }, []);
  return <div>{data?.name}</div>;
}
// DONT DO THIS BAD PATTERN 
function UserProfile() {
  const user = useUser();
  return <div>{user?.name}</div>;
}
// DONT DO THIS BAD PATTERN 
```

```react
// Better
function useUser() {
  const [data, setData] = useState(null);
  useEffect(() => {
    fetchUser().then(setData);
  }, []);
  return data;
}
function UserProfile() {
  const user = useUser();
  return <div>{user?.name}</div>;
}
```

## Key Architectural Concepts

- **App Router**: The application uses the Next.js App Router, with pages and layouts in `src/app/`.
- **Component-Based UI**: The UI is built with React components in `src/components/`. Generic UI components are in `src/components/ui/`, feature-specific components in `src/components/[feature]/`.
- **Server and Client Components**: The project uses a mix of React Server Components (RSC) and Client Components. Pay attention to the `'use client'` directive.
- **Database**: The database schema is defined in `src/lib/db/schema.ts` using Drizzle ORM. Database queries belong in `src/lib/db/`.
- **Authentication**: Authentication is handled with NextAuth.js. The configuration is in `src/lib/auth.ts` and the API route in `src/app/api/auth/`.
- **3D Model Handling**: The core 3D logic is in `src/components/models/` (e.g., `model-viewer.tsx`). This includes upload, rendering, and analysis.

## Development Workflow

- **Installation**: `pnpm install`
- **Development Server**: `pnpm dev`
- **Linting**: `pnpm lint`
- **Build**: `pnpm build`

## Coding Conventions

- **Imports**: Use the `@/` alias for imports from the `src` directory.
- **Styling**: Use Tailwind CSS utility classes. New UI components go in `src/components/ui/`.
- **State Management**: For client state, use React hooks (`useState`, `useReducer`, `useContext`). For server state, use Next.js App Router features like `searchParams` and Server Actions.
- **Data Fetching**: Use Next.js data fetching (e.g., in Server Components) or Server Actions.
- **Forms**: Use React Hook Form and Zod for forms and validation.
- **Database**: Use Drizzle Kit for schema migrations. After schema changes: `pnpm drizzle-kit generate`.



## Project Structure & File Naming

  - **File Names**: Always use **kebab-case** (e.g., `create-task-form.tsx`, `user-profile.ts`).
  - **Components**: UI components go in `src/components/ui/`, feature-specific components in `src/components/[feature]/`.
  - **Functions**: Function names must be in **camelCase** (e.g., `calculatePrice`, `getUserSession`).
  - **Types/Interfaces**: Use **PascalCase** (e.g., `UserProfile`, `ModelAnalysisResult`).
  - **Pages**: Each route in `src/app/` has its own folder containing a `page.tsx`, related actions in `actions.ts`, and any forms/components within the same directory.
    **Example**:
    ```
    /create-task/
      actions.ts
      create-task-form.tsx
      page.tsx

    /tasks/[taskId]/
      _components/
        task-view.tsx
      actions.ts
      page.tsx
    ```
    This keeps the logic and UI for a route bundled together.
  - **Import Alias**: `@/` → `src/`
  - **Styling**: Tailwind, with variables in `globals.css`.
  - **Error Boundaries**: Use for the 3D viewer and other critical UI elements.
  - **Auth Utils**: Server-side session helpers are in `auth-utils.ts`.

## Key Files and Directories

- `src/app/page.tsx`: Landing page
- `src/app/api/auth/`: NextAuth.js API handler
- `src/lib/auth.ts`: Auth configuration and utilities
- `src/lib/db/schema.ts`: Database schema
- `src/components/models/model-viewer.tsx`: 3D model viewer
- `src/components/models/price-calculator.tsx`: Price calculator
- `src/components/models/file-upload.tsx`: 3D model upload


## Excluded Files
- `.aiexclude`: Contains paths that should be ignored by the AI assistant.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/goldjunge91) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
