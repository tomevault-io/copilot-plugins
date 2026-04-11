## kreitzer-dot-dev

> `kreitzer-dot-dev` is a personal portfolio and blog website built with a modern web stack featuring Next.js 15, React 19, and TypeScript. It leverages the Next.js App Router for navigation and API handling.

# Project Overview

`kreitzer-dot-dev` is a personal portfolio and blog website built with a modern web stack featuring Next.js 15, React 19, and TypeScript. It leverages the Next.js App Router for navigation and API handling.

## Key Technologies

-   **Framework:** Next.js 15 (App Router)
-   **Language:** TypeScript
-   **Styling:** Tailwind CSS
-   **UI Library:** shadcn/ui (built on Radix UI), Lucide React (icons)
-   **Animations:** Framer Motion
-   **Content Management:** Markdown files with Frontmatter (stored in `content/blog/`)
-   **Markdown Processing:** `gray-matter`, `remark`, `remark-html`, `remark-gfm`, `remark-math`
-   **Diagrams:** Mermaid (processed via custom remark plugin)

## Architecture

-   **`app/`**: Contains the application routes, layouts, and API endpoints.
    -   `app/api/blog/`: RESTful endpoints for fetching blog data.
    -   `app/blog/[slug]/`: Dynamic route for individual blog posts.
-   **`content/blog/`**: The source of truth for blog posts. Each post is a Markdown file (`.md`) with YAML frontmatter.
-   **`components/`**: Reusable React components.
    -   `ui/`: Individual shadcn/ui components (e.g., `button.tsx`, `card.tsx`).
-   **`lib/`**: Utility functions.
    -   `blog.ts`: Server-side logic to read, parse, and process Markdown files. It handles `remark` plugins for math and Mermaid diagrams.
    -   `utils.ts`: Helper functions, including the `cn` utility for Tailwind class merging.
-   **`hooks/`**: Custom React hooks.

# Building and Running

## Scripts

The `package.json` defines the following primary scripts:

-   `npm run dev`: Starts the development server at `http://localhost:3000`.
-   `npm run build`: Builds the application for production. *Note: TypeScript and ESLint errors are currently ignored during the build process.*
-   `npm start`: Starts the production server.
-   `npm run lint`: Runs ESLint to check for code quality issues.

## Configuration

-   **`next.config.mjs`**: Configured to ignore TypeScript and ESLint errors during builds. Image optimization is disabled (`unoptimized: true`).
-   **`tsconfig.json`**: configured with the `@/*` alias pointing to the project root.
-   **Environment Variables**: The project uses `.env.local` for configuration (e.g., user details, social links).

# Development Conventions

-   **Path Aliases:** Use the `@/` alias for imports (e.g., `import { Button } from "@/components/ui/button"`).
-   **Styling:** Use Tailwind CSS classes for styling. Use the `cn()` utility when merging classes conditionally.
-   **Components:** Prefer using the existing components in `components/ui/`. If a new UI primitive is needed, follow the shadcn/ui pattern.
-   **Blog Posts:**
    -   New posts go in `content/blog/`.
    -   Must include frontmatter: `title`, `description`, `date`, `tags`, `published`.
    -   The `slug` is derived from the filename.
-   **Markdown:** Supports GitHub Flavored Markdown (GFM), LaTeX math equations, and Mermaid diagrams.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/markuskreitzer)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/markuskreitzer)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
