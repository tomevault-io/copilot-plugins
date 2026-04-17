## airoadmap

> *   Framework: Next.js (App Router structure primarily in `app/`, some legacy code might be in `client/src/`)

# .cursorrules - Project Guidelines for AI Assistant

## 1. Tech Stack & Dependencies

*   **Frontend:**
    *   Framework: Next.js (App Router structure primarily in `app/`, some legacy code might be in `client/src/`)
    *   Language: TypeScript
    *   UI Library: React
    *   Styling: Tailwind CSS
    *   Component Library: Shadcn UI (built on Radix UI)
    *   State Management: Likely Zustand or TanStack React Query (Confirm if one is primary)
    *   Routing: Next.js App Router (for `app/`), `react-router-dom` (for `client/src/`)
*   **Backend:**
    *   Runtime: Node.js
    *   Framework: Next.js API Routes (`app/api/.../route.ts`) are preferred. Legacy Express routes exist in `server/routes.ts`.
    *   Language: TypeScript
    *   Database: PostgreSQL
    *   ORM: Drizzle ORM (`drizzle.config.ts`)
    *   Storage Layer: Abstracted via `server/storage.ts` with a PostgreSQL implementation in `server/pg-storage.ts`.
*   **Shared:**
    *   Code shared between frontend and backend resides in the `shared/` directory.
    *   Schema definitions (including database types) are in `shared/schema.ts`.
*   **Modules:** Project uses ES Modules (`"type": "module"` in `package.json`). All new code should use ES Module syntax (`import`/`export`).

## 2. Project Structure

*   **`app/`**: Contains Next.js App Router pages, layouts, API routes, and related components. This is the preferred location for new frontend features and API endpoints.
*   **`client/src/`**: Contains legacy client-side code (React components, pages, hooks). Prefer adding new frontend code to the `app/` directory unless modifying existing legacy features.
*   **`server/`**: Contains backend-specific logic, including storage implementations (`pg-storage.ts`, `storage.ts`), legacy Express routes (`routes.ts`), database utilities, and potentially other server-side modules. API routes should primarily live in `app/api/`.
*   **`shared/`**: Contains code shared between the client and server, primarily TypeScript types, schemas (`schema.ts`), and shared utilities (`scoring.ts`).
*   **`components/`**: Likely contains reusable UI components used across `client/src` or potentially `app/`. Prefer organizing components colocated with features within `app/` where possible.
*   **`lib/`**: Contains general utility functions, potentially shared or client/server specific depending on location (e.g., `lib/scoring-ui.ts` for client UI).
*   **`hooks/`**: Contains custom React hooks, primarily for the `client/src` part of the application.

## 3. Key Principles & Architecture

*   **Next.js App Router First:** Prioritize using Next.js App Router features (Server Components, API Routes in `app/api/`) for new development.
*   **Minimize Client Components:** Use React Server Components (RSC) by default. Only use `'use client'` when interactivity or browser APIs are absolutely necessary.
*   **API Routes:** Prefer Next.js API Routes (`app/api/.../route.ts`) over the legacy Express routes in `server/routes.ts`.
*   **Database Interaction:** All database operations should go through the abstraction layer defined in `server/storage.ts` and implemented in `server/pg-storage.ts`. Use Drizzle ORM for database queries.
*   **Schema Definition:** The source of truth for data structures and database schema is `shared/schema.ts`.
*   **Type Safety:** Adhere to strict TypeScript practices. Use types defined in `shared/schema.ts` where applicable.
*   **Path Aliases:** Use defined path aliases from `tsconfig.json` for cleaner imports:
    *   `@/*` -> `./client/src/*` (Potentially needs update if `app/` is primary)
    *   `@shared/*` -> `./shared/*`
    *   `@/server/*` -> `./server/*`
    *   `@/app/*` -> `./app/*`

## 4. Coding Standards

*   Follow standard TypeScript best practices.
*   Use functional components with Hooks in React. Avoid class components.
*   Write clear, concise, and maintainable code.
*   Add JSDoc comments for complex functions and components.
*   Ensure code is formatted according to project standards (likely Prettier/ESLint configured).
*   Use ESM `import`/`export` syntax.

## 5. Naming Conventions

*   **Directories:** `lowercase-with-dashes` (e.g., `components/auth-wizard`).
*   **Files:** `PascalCase.tsx` for React components, `camelCase.ts` or `kebab-case.ts` for utility files. Be consistent.
*   **Variables/Functions:** `camelCase`.
*   **Types/Interfaces:** `PascalCase`.

## 6. Database

*   **Type:** PostgreSQL
*   **ORM:** Drizzle ORM
*   **Schema:** Defined in `shared/schema.ts` using Drizzle schema syntax.
*   **Access:** Via `server/storage.ts` and `server/pg-storage.ts`.

## 7. Build/Development

*   **Package Manager:** npm
*   **Development Server:** `npm run dev`
*   **Build:** `npm run build`

## 8. Deployment

*   **Production Environment:** Vercel (indicated by `vercel.json`)
*   **Development Environment:** Local machine (via `npm run dev`) 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/roadchimp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
