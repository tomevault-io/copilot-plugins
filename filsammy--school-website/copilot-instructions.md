## school-website

> This is a Next.js 14 project for the China Young Talent Association school website. It uses the App Router and TypeScript.

# Project Overview

This is a Next.js 14 project for the China Young Talent Association school website. It uses the App Router and TypeScript.

## Core Technologies

*   **Framework:** Next.js 14 (App Router)
*   **Language:** TypeScript
*   **Styling:** Tailwind CSS
*   **UI Components:** shadcn/ui ("new-york" style)
*   **Icons:** Lucide Icons
*   **Linting:** ESLint with Next.js specific configuration

## Building and Running

*   **Install Dependencies:** `npm install`
*   **Run Development Server:** `npm run dev` (available at http://localhost:3000)
*   **Build for Production:** `npm run build`
*   **Run Production Server:** `npm run start`
*   **Lint:** `npm run lint`

## Development Conventions

*   **Path Aliases:** The project uses path aliases defined in `tsconfig.json` and `components.json`. The `@/` alias maps to the `src/` directory.
*   **Component Structure:**
    *   Reusable UI components are located in `src/components/ui`.
    *   Layout components like the navbar and footer are in `src/components/layout`.
*   **Styling:** Tailwind CSS is used for styling. Global styles are in `src/app/globals.css`.
*   **State Management:** The `README.md` mentions plans to use TanStack Query (React Query) and Zod for state and data management.
*   **Forms:** The `README.md` mentions plans to use React Hook Form integrated with Zod.
*   **Authentication:** The `README.md` mentions plans to use NextAuth.js / Auth.js.
*   **Database:** The `README.md` mentions plans for a PostgreSQL database with Prisma.
*   **AI Integration:** The `README.md` has a placeholder for AI integration, with potential use cases like an AI school assistant chatbot.
*   **Deployment:** The `README.md` suggests Vercel for deployment.

## Key Files

*   **`src/app/layout.tsx`:** The root layout of the application. It sets up the HTML structure, fonts, and includes the main navigation bar.
*   **`src/app/page.tsx`:** The homepage of the website. It currently displays a video background with a "Get Started" button.
*   **`src/components/layout/navbar.tsx`:** The main navigation bar for the website. It's responsive and includes links to various pages and dropdown menus.
*   **`src/components/layout/footer.tsx`:** The footer for the website. This file is currently empty.
*   **`package.json`:** Defines the project's dependencies and scripts.
*   **`next.config.ts`:** The Next.js configuration file. It's currently using the default configuration.
*   **`tsconfig.json`:** The TypeScript configuration file. It defines the compiler options and path aliases.
*   **`postcss.config.mjs`:** The PostCSS configuration file. It's set up to use Tailwind CSS.
*   **`eslint.config.mjs`:** The ESLint configuration file. It uses the recommended Next.js configuration.
*   **`components.json`:** The configuration file for shadcn/ui.

---
> Source: [filsammy/school_website](https://github.com/filsammy/school_website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
