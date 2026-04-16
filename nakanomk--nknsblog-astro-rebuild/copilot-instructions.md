## nknsblog-astro-rebuild

> This file provides a comprehensive overview of the `Nknsblog_astro_rebuild` project, intended to serve as a quick-start guide and context for development.

# Project: Nknsblog Astro Rebuild

This file provides a comprehensive overview of the `Nknsblog_astro_rebuild` project, intended to serve as a quick-start guide and context for development.

## Project Overview

This is a personal blog website for "NknSのSitE" built using the [Astro](https://astro.build/) framework and the "Astro Theme Pure". The project is configured in TypeScript and styled with [UnoCSS](https://unocss.dev/). It's a highly customized setup with a focus on performance, typography, and rich content features.

### Key Technologies

*   **Framework**: Astro
*   **Theme**: Astro Theme Pure
*   **Language**: TypeScript
*   **Styling**: UnoCSS (with `presetMini` and `presetTypography`)
*   **Deployment**: Vercel (Serverless)
*   **Package Manager**: bun

### Core Features

*   **Content**: Markdown (MDX) with support for LaTeX math equations (`rehype-katex`) and advanced syntax highlighting (Shiki).
*   **Comments**: Integrated with [Waline](https://waline.js.org/) via the server at `https://waline.nkns.cc`.
*   **Search**: Full-site search is implemented using [Pagefind](https://pagefind.app/).
*   **Images**: Uses `sharp` for image optimization and `medium-zoom` for a lightbox effect on images.
*   **SEO**: The theme automatically generates a sitemap and RSS feed.
*   **Code Blocks**: Enhanced with custom Shiki transformers to add titles, language indicators, and a "copy" button.

## Building and Running

The project uses `bun` as the package manager.

*   **Install Dependencies**:
    ```shell
    bun install
    ```

*   **Run Development Server**:
    Starts a local development server with hot-reloading.
    ```shell
    bun dev
    ```

*   **Build for Production**:
    Builds the static site for deployment. This command also runs type checking and linting.
    ```shell
    bun run build
    ```

*   **Preview Production Build**:
    Starts a local server to preview the production build.
    
    ```shell
    bun preview
    ```
    
*   **Create a New Post**:
    The project includes a helper script to scaffold a new blog post.
    ```shell
    bun new
    ```

## Development Conventions

*   **Code Style**: Code formatting is enforced by [Prettier](https://prettier.io/). Code quality and rules are enforced by [ESLint](https://eslint.org/).
    *   **Format Code**: `bun run format`
    *   **Lint Code**: `bun run lint`

*   **Configuration**:
    *   **`astro.config.ts`**: Main Astro configuration file. Defines integrations, markdown settings, and the Vercel deployment adapter.
    *   **`src/site.config.ts`**: Theme-specific configuration. This is where you can change the site title, author, header menu, footer links, and configure integrations like Waline and Pagefind.
    *   **`uno.config.ts`**: UnoCSS configuration. Defines the color palette, typography rules, and other styling presets.
    *   **`tsconfig.json`**: TypeScript compiler options.

*   **Content Structure**:
    *   Blog posts are located in `src/content/blog/` as `.md` or `.mdx` files.
    *   Other pages (About, Links, etc.) are located in `src/pages/`.
    *   Layouts in `src/layouts/` define the structure for different page types.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Nakanomk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
