## designer-portfolio

> Generates TypeScript types from the Supabase project schema.

# Designer Portfolio - GEMINI.md

## Project Overview
This project is a personal portfolio for an experienced product designer. it features a public-facing home page and a password-protected dashboard for managing content such as projects, skills, and user information.

### Tech Stack
- **Framework:** React 19 (Vite / Rolldown)
- **Language:** TypeScript
- **Styling:** Tailwind CSS v4, shadcn-ui, Radix UI
- **Icons:** Lucide React
- **Backend:** Supabase (Authentication & PostgreSQL)
- **Data Fetching:** TanStack Query v5 (with persistence)
- **Forms:** React Hook Form + Zod
- **Routing:** React Router v7 (HashRouter)
- **Animation:** Motion (framer-motion)

## Building and Running

### Development
```bash
yarn dev
```
Starts the development server on [http://localhost:3000](http://localhost:3000).

### Build
```bash
yarn build
```
Builds the application for production using `tsc` and Vite.

### Database Types
```bash
yarn db:types
```
Generates TypeScript types from the Supabase project schema.

### Linting
```bash
yarn lint
```
Runs ESLint to check for code quality and style issues.

## Project Structure & Conventions

### Directory Layout
- `src/components/ui`: shadcn-ui base components.
- `src/components`: Application-specific components (e.g., `entity-table`, `skills-section`).
- `src/pages`: Page components organized by route (Home, Dashboard, Login).
- `src/remote`: Backend integration layer, including Supabase client, query providers, and hooks.
- `src/theme`: Theme configuration and provider (supporting dark/light modes).
- `src/lib`: Shared utilities and helpers.

### Development Conventions
- **Routing:** Uses `HashRouter` for compatibility with static hosting environments.
- **Backend Integration:** All Supabase interactions should go through `src/remote`. Use the `sb` export (Supabase client) and TanStack Query for data fetching.
- **Styling:** Adhere to Tailwind CSS v4 conventions. Use CSS variables defined in `src/styles.css` for consistent spacing, colors, and shadows.
- **Forms:** Use `react-hook-form` paired with `zod` schemas for validation.
- **Components:** Favor composition and small, reusable components. Use `clsx` and `tailwind-merge` for class manipulation.
- **Types:** Run `yarn db:types` after making schema changes in Supabase to keep `src/remote/database.types.ts` up to date.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vyndrix)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/vyndrix)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
