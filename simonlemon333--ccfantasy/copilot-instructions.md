## ccfantasy

> This is a fantasy football (soccer) web application named "ccfantasy". It is built with Next.js (App Router) and uses Supabase for its backend, including database (PostgreSQL), authentication, and real-time features. The frontend is styled with Tailwind CSS.

# GEMINI.md

## Project Overview

This is a fantasy football (soccer) web application named "ccfantasy". It is built with Next.js (App Router) and uses Supabase for its backend, including database (PostgreSQL), authentication, and real-time features. The frontend is styled with Tailwind CSS.

The core functionality allows users to create and join fantasy leagues (rooms), draft a team of players within a budget, and compete based on points scored in real-world football matches. The project has a detailed roadmap that includes plans for future Web3 integration, such as on-chain result verification and NFT-based rewards.

## Building and Running

The project uses `pnpm` as its package manager. The following commands are essential for development:

*   **Install dependencies:**
    ```bash
    pnpm install
    ```

*   **Run the development server:**
    ```bash
    pnpm dev
    ```
    The application will be available at [http://localhost:3000](http://localhost:3000).

*   **Build for production:**
    ```bash
    pnpm build
    ```

*   **Run the production server:**
    ```bash
    pnpm start
    ```

*   **Lint the code:**
    ```bash
    pnpm lint
    ```

*   **Type-check the code:**
    ```bash
    pnpm run type-check
    ```

## Development Conventions

*   **Framework:** The project uses Next.js with the App Router directory structure (`src/app`).
*   **Language:** The primary language is TypeScript.
*   **Styling:** Tailwind CSS is used for styling.
*   **Backend:** Supabase provides the backend services. Client-side interaction with Supabase is handled in `src/lib/supabase.ts` and authentication logic is in `src/lib/auth.ts`.
*   **Database:** The database schema is managed through SQL scripts located in the `scripts/` directory.
*   **API:** API routes are implemented as Next.js Route Handlers within the `src/app/api/` directory.
*   **Code Quality:** The project is set up with ESLint for linting and `commitlint` for enforcing commit message conventions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/simonlemon333)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/simonlemon333)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
