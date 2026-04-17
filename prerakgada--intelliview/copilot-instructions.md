## intelliview

> This is a Next.js application for **IntelliView**, an intelligent platform for job interviewing and candidate management. It leverages a modern tech stack to provide a comprehensive solution for recruiters, hiring managers, and candidates.

# GEMINI.md

## Project Overview

This is a Next.js application for **IntelliView**, an intelligent platform for job interviewing and candidate management. It leverages a modern tech stack to provide a comprehensive solution for recruiters, hiring managers, and candidates.

The application is built with the following technologies:

*   **Framework:** Next.js (App Router)
*   **Language:** TypeScript
*   **Styling:** Tailwind CSS with shadcn/ui components
*   **Authentication:** Supabase
*   **Database:** PostgreSQL with Prisma ORM
*   **State Management:** Redux Toolkit with redux-persist
*   **Validation:** Zod

The architecture includes a client-side `AuthGuard` for role-based access control, server-side actions for data fetching, and a well-defined database schema to manage the core entities of the application.

## Building and Running

To get the project up and running, follow these steps:

1.  **Install dependencies:**
    ```bash
    npm install
    ```

2.  **Set up environment variables:**
    *   Copy `.env.example` to `.env.local`.
    *   Fill in the required values for `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, and `DATABASE_URL`.

3.  **Generate Prisma client:**
    ```bash
    npx prisma generate
    ```

4.  **Run database migrations:**
    ```bash
    npx prisma migrate dev
    ```

5.  **Start the development server:**
    ```bash
    npm run dev
    ```

### Key Scripts

*   `npm run dev`: Starts the development server.
*   `npm run build`: Creates a production build of the application.
*   `npm run start`: Starts the production server.
*   `npm run lint`: Lints the codebase using ESLint.
*   `npm run type-check`: Checks for TypeScript errors.

## Development Conventions

*   **Authentication:** User authentication is handled by Supabase. A client-side `AuthGuard` component (`src/components/auth-guard.tsx`) is used to protect routes and implement role-based access control.
*   **Data Fetching:** Server-side data fetching is performed using server actions located in `src/actions`. These actions use Prisma to interact with the database.
*   **Database:** The database schema is defined in `prisma/schema.prisma`. All database models and their relations are specified in this file.
*   **State Management:** Global application state is managed using Redux Toolkit. The Redux store is configured in `src/store`, and slices for different features are located in `src/store/slices`.
*   **Styling:** The application uses Tailwind CSS for styling, with UI components from shadcn/ui. Custom components are located in `src/components`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PrerakGada) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
