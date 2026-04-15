## sniperok

> This document provides instructions for Gemini to assist with development tasks for the **sniperok** project.

# Gemini Project Helper

This document provides instructions for Gemini to assist with development tasks for the **sniperok** project.

## Project Overview

**sniperok** is a web application built with SvelteKit, utilizing Supabase for its backend and database services. The application is written in TypeScript.

## Key Technologies

- **Framework**: [SvelteKit](https://kit.svelte.dev/)
- **Language**: [TypeScript](https://www.typescriptlang.org/)
- **Backend & DB**: [Supabase](https://supabase.com/)
- **Database Client**: [Kysely](https://kysely.dev/) (for type-safe SQL queries)
- **Styling**: [Tailwind CSS](https://tailwindcss.com/) with `tailwind-variants`
- **UI Components**: [shadcn-svelte](https://www.shadcn-svelte.com/) / [bits-ui](https://www.bits-ui.com/)
- **Icons**: [Lucide Svelte](https://lucide.dev/guide/packages/lucide-svelte)
- **Forms**: [SvelteKit Superforms](https://superforms.rocks/)
- **Schema Validation**: [Zod](https://zod.dev/)
- **Testing**: [Vitest](https://vitest.dev/)
- **Linting**: [ESLint](https://eslint.org/)
- **Formatting**: [Prettier](https://prettier.io/)

## Development Setup & Commands

Before running the application, install the dependencies:

```bash
pnpm install
```

The following scripts are available in `package.json`:

- `pnpm run dev`: Starts the development server with live reloading.
- `pnpm run build`: Compiles the application for production.
- `pnpm run preview`: Serves the production build locally.
- `pnpm run test`: Runs the test suite using Vitest.
- `pnpm run lint`: Lints the codebase using ESLint.
- `pnpm run format`: Formats the code using Prettier.
- `pnpm run check`: Runs static type checking using `svelte-check`.
- `pnpm run db:reset`: Resets the local Supabase database and runs seed scripts.
- `pnpm run db:codegen`: Generates TypeScript types for the database schema using Kysely. Run this after any database migration.

## Coding Style and Conventions

- **Always use TypeScript**.
- **Follow existing patterns**: When adding new features, adhere to the coding style and structure found in the existing codebase.
- **Components**: Create reusable components in `src/lib/components`. For UI primitives, use `bits-ui` and style them with Tailwind CSS.
- **State Management**: For page-level state, use Svelte stores. For server data, use SvelteKit's `load` functions.
- **Forms**: Use `SvelteKit Superforms` along with `Zod` for schema definition and validation.
- **Database**: All database interactions must be type-safe using the Kysely client. After any schema change (`supabase/migrations`), you **must** run `pnpm run db:codegen` to update the TypeScript types.
- **API Routes**: Create API endpoints as `+server.ts` files within the `src/routes` directory.

## Workflow Preferences

- **Work in small increments**: don;t make sweeping changes in one go, try to break it into manageble pieces.
  Formulate a plan, and show the user get agreement to move ahead. Commit work before moving onto the next change.
- **Prioritize Linting and Formatting**: Always address and fix linting and formatting issues before implementing new features or making significant functional changes. This ensures a clean and consistent codebase throughout the development process.

## Commit Workflow

To ensure consistent and reliable commits, use the following method:

1. Stage your changes using `git add <file(s)>`.
2. Create a commit message file (e.g., `temp/commit_message.txt`) with the desired message.
3. Commit using `git commit -F temp/commit_message.txt`.

## Directory Structure

- `src/routes`: Contains all pages and API endpoints for the application.
- `src/lib`: Shared modules, utilities, and components.
    - `src/lib/components`: Reusable Svelte components.
        - `src/lib/components/ui`: Low-level UI primitives from `bits-ui`.
    - `src/lib/server`: Server-side code, including database access.
        - `src/lib/server/db`: Kysely instance, database types, and repositories.
- `supabase`: Supabase project configuration.
    - `supabase/migrations`: Database schema migrations.
    - `supabase/seed.sql`: Seed data for the database.
- `static`: Static assets like images and fonts.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/swopsyapp)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/swopsyapp)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
