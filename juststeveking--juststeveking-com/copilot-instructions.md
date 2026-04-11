## juststeveking-com

> This is a personal website and blog built with [Astro](https://astro.build/). It uses [Tailwind CSS](https://tailwindcss.com/) for styling and [MDX](https://mdxjs.com/) for content. The site is deployed to Cloudflare Pages.

# Project Overview

This is a personal website and blog built with [Astro](https://astro.build/). It uses [Tailwind CSS](https://tailwindcss.com/) for styling and [MDX](https://mdxjs.com/) for content. The site is deployed to Cloudflare Pages.

The project is structured around Astro's content collections, which are used to manage articles, testimonials, videos, and more. Each collection has a defined schema using `zod` for validation.

## Building and Running

To work with this project, you'll need to have Node.js and pnpm installed.

**Installation:**

```bash
pnpm install
```

**Development:**

To start the development server, run:

```bash
pnpm run dev
```

This will start a local server at `http://localhost:4321`.

**Building:**

To build the site for production, run:

```bash
pnpm run build
```

The output will be in the `dist/` directory.

**Deployment:**

The site is deployed to Cloudflare Pages. The following commands are used for deployment:

```bash
# Deploy to production
pnpm run deploy

# Deploy a preview branch
pnpm run deploy:preview
```

## Development Conventions

*   **Content:** All content is managed in the `src/content/` directory. Each subdirectory corresponds to a content collection.
*   **Styling:** The project uses Tailwind CSS for styling. Utility classes are preferred over custom CSS.
*   **Components:** Reusable UI components are located in the `src/components/` directory.
*   **Linting and Formatting:** The project uses Prettier and ESLint for code formatting and linting. (Inferred from common practices, but not explicitly confirmed from the files.)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JustSteveKing)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/JustSteveKing)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
