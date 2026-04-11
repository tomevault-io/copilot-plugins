## astro-blog

> This document provides a complete technical overview of the Astro/React Static Blog project. It details the architecture, specifications, build process, and key conventions used throughout the codebase.

# GEMINI.md: Astro/React Static Blog

This document provides a complete technical overview of the Astro/React Static Blog project. It details the architecture, specifications, build process, and key conventions used throughout the codebase.

## 1. Project Overview

This project is a high-performance, minimalist static blog built according to the specifications outlined in `PRD.md`. It uses the Astro framework as its core, with React for interactive UI components ("islands") and Tailwind CSS for styling.

### Core Features:

- **Static Site Generation:** The entire site is pre-rendered to static HTML, CSS, and JavaScript at build time.
- **Content Collections:** Blog posts are managed as local Markdown (`.md`) and MDX (`.mdx`) files within `src/content/posts/`.
- **Dynamic Pages:** Astro's file-based routing generates individual post pages, a paginated blog index, and pages for tag-based filtering.
- **Interactive UI:** A client-side React component provides a dark mode toggle with `localStorage` persistence.
- **Automatic Sitemap:** An XML sitemap is generated automatically during the build process.

---

## 2. Architecture

The architecture is designed to be simple, performant, and maintainable, adhering to Astro's best practices.

### 2.1. Technology Stack

- **Framework:** Astro `4.x`
- **UI Components:** React `18.x`
- **Styling:** Tailwind CSS `3.x`
- **Content:** Astro Content Collections (Markdown & MDX)
- **Package Manager:** `npm`
- **Language:** TypeScript

### 2.2. Directory Structure

The project follows a standard Astro project structure, organized for clarity:

```
/
├── dist/                   # Build output directory
├── public/                 # Static assets (favicon, images)
├── src/
│   ├── components/
│   │   ├── astro/          # Reusable Astro components (PostCard.astro, TOC.astro)
│   │   └── react/          # Interactive React components (DarkModeToggle.tsx)
│   ├── content/
│   │   ├── config.ts       # Schema definition for content collections
│   │   └── posts/          # Blog post entries (.md, .mdx)
│   ├── layouts/
│   │   ├── BaseLayout.astro # Global site template
│   │   └── PostLayout.astro # Layout for individual blog posts
│   ├── pages/
│   │   ├── blog/
│   │   │   ├── [...page].astro # Paginated blog index
│   │   │   ├── [slug].astro    # Individual post pages
│   │   │   └── tags/
│   │   │       └── [tag].astro   # Tag-filtered post list
│   │   └── index.astro         # Homepage
│   └── styles/
│       └── global.css          # Global styles (contains Tailwind directives)
├── astro.config.mjs        # Astro configuration file
├── package.json
├── README.md
└── tsconfig.json
```

### 2.3. Key Components & Logic

- **Content Collections (`src/content/`):** This is the "headless CMS" of the project. The schema in `config.ts` enforces type safety and validation for all post frontmatter, ensuring consistency.

- **Layouts (`src/layouts/`):**
  - **`BaseLayout.astro`:** Serves as the master template. It defines the global `<html>`, `<head>`, and `<body>` structure, and includes the `<header>` and `<footer>`. The `DarkModeToggle` React component is rendered here with a `client:load` directive to make it immediately interactive.
  - **`PostLayout.astro`:** A wrapper for individual blog posts that uses `BaseLayout`. It programmatically displays post metadata from frontmatter (title, date, tags, etc.) and provides a `<slot />` for the main post content and Table of Contents.

- **Pages (`src/pages/`):**
  - **`blog/[...page].astro`:** Implements pagination. It uses `getStaticPaths` with Astro's `paginate` function on the 'posts' collection to generate paginated index pages (e.g., `/blog/`, `/blog/2/`, etc.).
  - **`blog/[slug].astro`:** Renders a single blog post. It uses `getStaticPaths` to create a route for every post. The `post.render()` method provides both the post's `Content` and its `headings` array, which is passed to the `TOC.astro` component.
  - **`blog/tags/[tag].astro`:** Creates a page for each unique tag. `getStaticPaths` is used to find all unique tags across all posts and generate corresponding routes. The page then filters and displays only the posts containing that tag.

- **Components (`src/components/`):**
  - **`PostCard.astro`:** A reusable component for displaying a post summary on the index and tag pages.
  - **`TOC.astro`:** Renders the Table of Contents by iterating over the `headings` array passed from `[slug].astro`.
  - **`DarkModeToggle.tsx`:** A client-side interactive React component. It checks `localStorage` and `prefers-color-scheme` to set the initial state, toggles the `.dark` class on the `<html>` element, and persists the user's choice.

---

## 3. Build & Deployment Process

### 3.1. Setup

1.  **Install Dependencies:** Ensure Node.js and `npm` are installed. Clone the repository and run:
    ```bash
    npm install
    ```

### 3.2. Development

2.  **Run the Dev Server:** To start the local development server with hot-reloading:
    ```bash
    npm run dev
    ```
    The site will be available at `http://localhost:4321`.

### 3.3. Production Build

3.  **Build the Site:** To generate the static, production-ready site:
    ```bash
    npm run build
    ```
    This command compiles the entire application into the `dist/` directory. All assets are optimized and bundled.

### 3.4. Deployment

4.  **Deploy:** The contents of the generated `dist/` directory can be deployed to any static web hosting provider (e.g., Vercel, Netlify, GitHub Pages).

---

## 4. Specifications & Conventions

- **Content Schema:** All blog posts in `src/content/posts/` must adhere to the schema defined in `src/content/config.ts`. Key frontmatter fields include `title`, `description`, `pubDate`, `heroImage` (optional), and `tags` (optional).
- **Styling:** All styling is handled by Tailwind CSS. The `darkMode: 'class'` strategy is enabled in `tailwind.config.cjs`.
- **Sitemap:** The `@astrojs/sitemap` integration automatically generates a sitemap on build, based on the `site` URL configured in `astro.config.mjs`.
- **Interactivity:** Client-side interactivity is handled exclusively by React components rendered with an appropriate `client:*` directive.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/syafiqfaiz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/syafiqfaiz)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
