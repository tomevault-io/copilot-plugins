## comercila-app

> This is a comprehensive business management application built with Next.js for supermarket operations, sales tracking, inventory management, and order processing. The application uses Supabase for its backend, Tailwind CSS for styling, and Recharts for data visualization. The frontend is built with React and TypeScript, and it follows a component-based architecture with a strong emphasis on custom hooks for state management and business logic.

# Project: Comercila - Business Management Dashboard

## Project Overview

This is a comprehensive business management application built with Next.js for supermarket operations, sales tracking, inventory management, and order processing. The application uses Supabase for its backend, Tailwind CSS for styling, and Recharts for data visualization. The frontend is built with React and TypeScript, and it follows a component-based architecture with a strong emphasis on custom hooks for state management and business logic.

The application has undergone a major refactoring to improve maintainability and code organization, resulting in a much cleaner and more efficient codebase.

## Building and Running

### Prerequisites

- Node.js and npm (or yarn/pnpm/bun) installed.
- Supabase project set up with the required schema (see `schema.sql`).
- Environment variables for Supabase properly configured in a `.env.local` file (see `.env.example` if available, or `src/lib/supabase.ts` for required variables).

### Key Commands

- **Development:** `npm run dev`
  - Runs the application in development mode with Turbopack.
  - Open [http://localhost:3000](http://localhost:3000) to view it in the browser.

- **Build:** `npm run build`
  - Builds the application for production.

- **Start:** `npm run start`
  - Starts a production server.

- **Lint:** `npm run lint`
  - Lints the codebase for potential errors and style issues.

## Development Conventions

- **Component-Based Architecture:** The application is built around reusable React components, located in `src/components`.
- **Custom Hooks:** State management and business logic are encapsulated in custom hooks, found in `src/hooks`.
- **Utility Functions:** Pure, testable functions for business logic and data transformations are located in `src/utils`.
- **Styling:** Tailwind CSS is used for styling, with the configuration in `tailwind.config.js`.
- **Error Handling:** The application uses error boundaries to catch and handle errors gracefully.
- **Performance:** Performance is a key consideration, with features like lazy loading, code splitting, and webpack optimizations implemented.
- **Testing:** The project has a focus on testing, with a combination of unit and integration tests.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/eXemoumen)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/eXemoumen)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
