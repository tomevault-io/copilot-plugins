## bun-line-better

> This document provides a high-level overview of the `bun-line-better` project, its architecture, and key technologies.

# Project Overview: bun-line-better

This document provides a high-level overview of the `bun-line-better` project, its architecture, and key technologies.

## Architecture

The project is structured as a monorepo managed by **Turborepo** and **Bun Workspaces**. This setup contains two primary applications:

-   `apps/server`: A backend server responsible for API logic, database interactions, and authentication.
-   `apps/web`: A frontend web application that provides the user interface.

## Tech Stack

### Core Technologies

-   **Runtime:** [Bun](https://bun.sh/) is used as the primary JavaScript runtime, package manager, and bundler across the project.
-   **Monorepo Manager:** [Turborepo](https://turbo.build/repo) is used to manage the workspaces and optimize build and development workflows.
-   **Language:** [TypeScript](https://www.typescriptlang.org/) is used for static typing in both the frontend and backend.
-   **Code Quality:** [Biome](https://biomejs.dev/) is used for linting and formatting the codebase to ensure consistency.

### Backend (`apps/server`)

-   **Web Framework:** [Hono](https://hono.dev/) is the web framework used to build the API, chosen for its lightweight and fast nature.
-   **Database ORM:** [Prisma](https://www.prisma.io/) is used to manage database schemas, migrations, and provide a type-safe query client.
-   **Authentication:** The `better-auth` library is implemented to handle user authentication and session management.
-   **Validation:** [Zod](https.zod.dev) is used for schema declaration and validation to ensure data integrity.

### Frontend (`apps/web`)

-   **Framework:** [React](https://react.dev/) is the core library for building the user interface.
-   **Build Tool:** [Vite](https://vitejs.dev/) provides a fast and modern development and build experience.
-   **Routing:** [TanStack Router](https://tanstack.com/router) manages client-side routing and navigation.
-   **Data Fetching & State:** [TanStack Query](https://tanstack.com/query) handles server-state management, including data fetching, caching, and synchronization.
-   **Forms:** [TanStack Form](https://tanstack.com/form) is used for building and managing forms with type-safety.
-   **Styling:** [Tailwind CSS](https://tailwindcss.com/) is the utility-first CSS framework used for styling.
-   **UI Components:** The UI is built upon a collection of components using:
    -   [Radix UI](https://www.radix-ui.com/) for unstyled, accessible primitives.
    -   A pattern similar to `shadcn/ui`, with custom components located in `src/components/ui`.
    -   `lucide-react` for icons.
    -   `sonner` for toast notifications.
-   **Theming:** `next-themes` is used to manage light and dark mode themes.

## Getting Started

-   **Install dependencies:** `bun install`
-   **Run development servers:** `bun dev`
-   **Build all apps:** `bun build`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Sh0ckWaveZero)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Sh0ckWaveZero)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
