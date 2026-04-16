## omedfrontendfigma

> This document provides a comprehensive overview of the OMED School Ideas Platform project, to be used as instructional context for future interactions.

# GEMINI.md - OMED School Ideas Platform

This document provides a comprehensive overview of the OMED School Ideas Platform project, to be used as instructional context for future interactions.

## Project Overview

OMED is a Reddit-style idea submission and voting platform designed for schools. It allows students to post ideas, and teachers and principals to review, manage, and approve them. The application is built as a modern web application with a clear path to be packaged for native mobile deployment on iOS and Android using Capacitor.

### Key Technologies

*   **Frontend Framework:** React 18 with TypeScript
*   **Build Tool:** Vite
*   **Styling:** Tailwind CSS with Shadcn/UI components and Lucide React for icons.
*   **Backend:** Supabase (PostgreSQL, Authentication, Realtime DB, Storage)
*   **Routing:** React Router v6
*   **UI Components:** A combination of custom components and primitives from Radix UI, organized under `src/components/ui`.

### Architecture

*   The application uses a component-based architecture, with pages located in `src/pages` and reusable components in `src/components`.
*   Authentication is managed via `src/contexts/AuthContext.tsx`, which provides user and session information throughout the app.
*   Routing is centralized in `src/App.tsx` and uses a `ProtectedRoute` component to enforce role-based access control (RBAC) for different user types (student, teacher, principal, admin).
*   The connection to the Supabase backend is handled by a client initialized in `src/lib/supabaseClient.ts`.

## Building and Running

### Prerequisites

*   Node.js v18+
*   An active Supabase account with a configured project (see `SUPABASE_SETUP.md` for details).

### Development

1.  **Install Dependencies:**
    ```bash
    npm install
    ```

2.  **Run the Development Server:**
    ```bash
    npm run dev
    ```
    The application will be available at `http://localhost:3000`.

### Production

1.  **Build the Application:**
    ```bash
    npm run build
    ```
    This command compiles the application and outputs the static files to the `/build` directory.

### Testing & Linting

*   **TODO:** No explicit `lint` or `test` scripts were found in `package.json`. It is recommended to add scripts for static analysis and unit/integration testing.
    *   A linting script could be: `"lint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0"`
    *   A type-checking script could be: `"typecheck": "tsc --noEmit"`

## Development Conventions

*   **Path Aliases:** The project uses the `@/` alias to refer to the `src/` directory for cleaner import paths. This is configured in `vite.config.ts`.
*   **Styling:** The project uses Tailwind CSS for utility-first styling. The configuration is in `tailwind.config.js`. Components are styled using a combination of utility classes and `class-variance-authority` for variants.
*   **UI Components:** The UI is built using components from the `shadcn/ui` library. These are not installed as a package, but rather the source code is added directly to the `src/components/ui` directory, allowing for full control and customization.
*   **Backend Interaction:** All interactions with the Supabase backend should go through the `supabase` client instance exported from `src/lib/supabaseClient.ts`. For organized code, it's recommended to create service files (e.g., `src/lib/ideaService.ts`) for different data models.
*   **Environment Variables:** Supabase credentials (`projectId` and `publicAnonKey`) are managed in `src/utils/supabase/info.tsx`. For better security and flexibility, these should be moved to `.env` files (e.g., `.env.local`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/OMED-school) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
