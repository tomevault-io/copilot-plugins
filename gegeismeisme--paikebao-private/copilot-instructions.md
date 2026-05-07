## paikebao-private

> This is a Next.js project called "keshibao" (课时宝), which appears to be a one-stop intelligent course scheduling solution.

# Project Overview

This is a Next.js project called "keshibao" (课时宝), which appears to be a one-stop intelligent course scheduling solution.

It is built with the following technologies:
- **Framework:** [Next.js](https://nextjs.org/) (using the App Router)
- **Language:** [TypeScript](https://www.typescriptlang.org/)
- **Styling:** [Tailwind CSS](https://tailwindcss.com/)
- **UI Components:** Likely [shadcn/ui](https://ui.shadcn.com/), based on the Tailwind CSS configuration and component structure.
- **Backend:** [Supabase](https://supabase.com/) for backend services and database.

The project is structured as a standard Next.js application. The main pages are located in `src/app`, and reusable UI components are in `src/components`.

# Building and Running

To get the project running locally, you will need to have Node.js and npm installed.

1.  **Install dependencies:**
    ```bash
    npm install
    ```

2.  **Set up environment variables:**
    This project uses Supabase, so you will need to create a `.env.local` file in the root of the project and add your Supabase URL and anon key:
    ```
    NEXT_PUBLIC_SUPABASE_URL=your-supabase-url
    NEXT_PUBLIC_SUPABASE_ANON_KEY=your-supabase-anon-key
    ```

3.  **Run the development server:**
    ```bash
    npm run dev
    ```
    This will start the development server on [http://localhost:3000](http://localhost:3000).

4.  **Build for production:**
    ```bash
    npm run build
    ```

5.  **Run in production mode:**
    ```bash
    npm run start
    ```

# Development Conventions

- **Linting:** The project uses ESLint for code linting. You can run the linter with:
  ```bash
  npm run lint
  ```
- **Component Structure:** UI components are built using Radix UI primitives and styled with Tailwind CSS, which is a common pattern for `shadcn/ui`.
- **Backend Interaction:** The application interacts with Supabase using the `@supabase/ssr` and `@supabase/supabase-js` libraries. The Supabase client is configured in `src/lib/supabase/client.ts`.

---
> Source: [gegeismeisme/paikebao_private](https://github.com/gegeismeisme/paikebao_private) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
