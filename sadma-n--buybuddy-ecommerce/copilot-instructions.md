## buybuddy-ecommerce

> This file provides a comprehensive overview of the BuyBuddy-Ecommerce project, its structure, and how to work with it.

# GEMINI.md

This file provides a comprehensive overview of the BuyBuddy-Ecommerce project, its structure, and how to work with it.

## Project Overview

BuyBuddy-Ecommerce is a full-stack e-commerce application built with a modern technology stack. It leverages the power of [TanStack](https://tanstack.com/) libraries for routing, data fetching, and more. The project is built with [React](https://react.dev/) and [TypeScript](https://www.typescriptlang.org/), and uses [Vite](https://vitejs.dev/) as its build tool.

### Key Technologies

*   **Frontend:** React, TypeScript, Vite, Tailwind CSS
*   **Routing:** TanStack Router
*   **Data Fetching:** TanStack Query
*   **Database:** PostgreSQL
*   **ORM:** Drizzle ORM
*   **Package Manager:** Bun

## Building and Running

### Prerequisites

*   [Bun](https://bun.sh/)
*   [Node.js](https://nodejs.org/en) (for `drizzle-kit`)
*   [PostgreSQL](https://www.postgresql.org/)

### Installation

```bash
bun install
```

### Running the Development Server

```bash
bun run dev
```

This will start the development server on `http://localhost:3000`.

### Building for Production

```bash
bun run build
```

This will create a production-ready build in the `.output` directory.

### Testing

```bash
bun run test
```

## Development Conventions

### Linting and Formatting

This project uses ESLint and Prettier for code linting and formatting.

*   To run the linter: `bun run lint`
*   To format the code: `bun run format`
*   To check for both linting and formatting errors: `bun run check`

### Database Migrations

The project uses Drizzle ORM for database management.

*   To generate a new migration: `bun run db:generate`
*   To apply migrations: `bun run db:migrate`
*   To push schema changes directly to the database (for development): `bun run db:push`
*   To open Drizzle Studio: `bun run db:studio`

### Environment Variables

The project uses T3Env for type-safe environment variables. Environment variables should be added to the `.env` or `.env.local` files and defined in `src/env.ts`.

### Project Structure

*   `src/routes`: Contains the file-based routes for the application.
*   `src/components`: Contains reusable React components.
*   `src/db`: Contains the database schema and Drizzle ORM configuration.
*   `src/lib`: Contains utility functions.
*   `src/hooks`: Contains custom React hooks.
*   `src/contexts`: Contains React context providers.
*   `src/stores`: Contains Zustand stores for state management.
*   `src/types`: Contains TypeScript type definitions.
*   `drizzle`: Contains the generated database migrations.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SADMA-N)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/SADMA-N)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
