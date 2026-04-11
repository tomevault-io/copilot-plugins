## astro-tailwind

> This project is a static site generator built with **Astro**, styled using **Tailwind CSS v4**, and maintained with **Biome** for linting and formatting. It serves as a modern, performance-focused web application foundation.

# Project Context: Astro + Tailwind CSS v4

## 1. Project Overview

This project is a static site generator built with **Astro**, styled using **Tailwind CSS v4**, and maintained with **Biome** for linting and formatting. It serves as a modern, performance-focused web application foundation.

### Key Technologies
*   **Framework:** [Astro](https://astro.build) (v5.x)
*   **Styling:** [Tailwind CSS v4](https://tailwindcss.com) (via `@tailwindcss/vite`)
*   **Linting & Formatting:** [Biome](https://biomejs.dev)
*   **Package Manager:** `pnpm`
*   **Icons:** [Lucide](https://lucide.dev/guide/packages/lucide-astro) (`@lucide/astro`)

### Using MCP Servers for Latest Documentation

**IMPORTANT**: When writing or modifying code, ALWAYS use the available MCP servers to verify current best practices and syntax:

*   **Astro MCP Server** (`mcp_astro_mcp_search_astro_docs`): Verify Astro component patterns, props, layouts, routing, and API usage.
*   **Tailwind CSS MCP Server** (`mcp_tailwindcss_m_search_tailwind_docs`, `mcp_tailwindcss_m_get_tailwind_utilities`): Verify utility classes, configuration, and v4-specific features.

**When to use:**
1.  Before implementing new Astro components or layouts.
2.  When uncertain about Tailwind CSS class names or configuration.
3.  When implementing file-based routing or dynamic routes.
4.  Before using Astro APIs like `Astro.props`, `Astro.glob()`, or `getStaticPaths()`.
5.  When troubleshooting or debugging issues.

## 2. Building and Running

All commands are run via `pnpm`.

### Core Commands
*   **Install Dependencies:** `pnpm install`
*   **Start Development Server:** `pnpm dev` (Runs on `http://localhost:4321`)
*   **Build for Production:** `pnpm build` (Output to `./dist/`)
*   **Preview Production Build:** `pnpm preview`

### Mise Task Shortcuts
*   `mise run astro:dev` - Start dev server
*   `mise run astro:build` - Build project (with confirmation)
*   `mise run astro:preview` - Preview build
*   `mise run astro:upgrade` - Upgrade Astro
*   `mise run astro:check` - Run Astro TypeScript check
*   `mise run biome:format` - Format src/
*   `mise run biome:lint` - Lint src/

### Quality Assurance
*   **Lint Code:** `pnpm lint` (Runs `biome lint --write`)
*   **Format Code:** `pnpm format` (Runs `biome format --write`)
*   **Type Check:** `pnpm check` (Runs `astro check`)

## 3. Development Conventions

### File Structure
*   `src/pages/`: File-based routing. Each `.astro` file becomes a route.
*   `src/components/`: Reusable UI components.
*   `src/layouts/`: Wrapper components for pages (e.g., `Layout.astro` defines `<head>` and `<body>`).
*   `src/styles/`: Global styles (e.g., `global.css` for font imports and Tailwind directives).
*   `src/assets/`: Optimized assets (images, SVGs).

### Design System Rules

#### Typography
*   **Font Family:** `Noto Sans JP` (Imported in `src/styles/global.css`).
*   **Base Style:** Applied globally in `src/layouts/Layout.astro`:
    ```css
    html, body {
      font-family: "Noto Sans JP", sans-serif;
    }
    ```

#### Styling (Tailwind CSS v4)
*   **Configuration:** Uses zero-config Tailwind v4. No `tailwind.config.js`.
*   **Directives:** `@import "tailwindcss";` in `src/styles/global.css`.
*   **Approach:** Utility-first. Use classes directly in markup.
*   **Responsive:** Use standard prefixes (`md:`, `lg:`, etc.).

#### Icons
*   **Library:** `@lucide/astro`.
*   **Usage:** Import components directly.
    ```astro
    import { RocketIcon } from '@lucide/astro';
    <RocketIcon class="w-6 h-6 text-blue-500" />
    ```

### Coding Style (Biome)
*   **Indentation:** 2 spaces.
*   **Quotes:** Single quotes for JavaScript/TypeScript.
*   **Semicolons:** As needed (avoid excessive usage where automatic insertion works safely, but follow Biome's specific rules).
*   **Imports:** Organize imports is disabled (`organizeImports: "off"`).

### Component Architecture
*   **Astro Components:** Prefer `.astro` files.
    *   **Frontmatter:** TypeScript interface for Props.
    *   **Template:** HTML with JSX-like expressions.
    *   **Styles:** Scoped `<style>` blocks are permitted but prefer Tailwind utilities.

### Common Pitfalls
* **Don't** use PostCSS config files - Tailwind v4 uses Vite plugin only
* **Don't** use old Tailwind import syntax (`@tailwind base;` etc.) - use `@import "tailwindcss"` in CSS files
* **Don't** use arbitrary values excessively (e.g., `text-[2rem]`) - prefer standard utilities in v4
* **Don't** import Lucide from `lucide-astro` or `lucide-react` - ALWAYS use `@lucide/astro` (the correct package for this project)
* **Don't** run ESLint/Prettier - use Biome commands exclusively
* **Don't** add semicolons everywhere - Biome uses `asNeeded` mode
* **Don't** import Google Fonts in Layout - already imported in `global.css`
* **Don't** organize imports automatically - Biome has this disabled

## 4. Configuration Files
*   **`astro.config.mjs`:** Main Astro config. Includes `tailwindcss()` Vite plugin.
*   **`biome.json`:** Rules for linter and formatter.
*   **`package.json`:** Scripts and dependencies.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gaspanik)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gaspanik)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
