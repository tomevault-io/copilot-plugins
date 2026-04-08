## formula-code-github-io

> Generates a static build in the `build/` directory.

# Project Overview

This project is a SvelteKit-based interactive web application, likely designed for data-driven storytelling or a visual presentation related to "Formula Code" and wine analysis. It leverages the "Svelte Starter" template from The Pudding, incorporating features for scrollytelling, data visualization, and content management via Google Docs.

**Key Technologies:**

*   **Framework:** [SvelteKit](https://kit.svelte.dev/) (using `@sveltejs/adapter-static` for static site generation).
*   **Bundler:** [Vite](https://vitejs.dev/).
*   **Visualization:** [D3.js](https://d3js.org/) and [LayerCake](https://layercake.graphics/) for charts and graphics.
*   **Content Management:** [ArchieML](http://archieml.org/) for integrating content from Google Docs and Sheets.
*   **Styling:** PostCSS with Autoprefixer, and [Style Dictionary](https://amzn.github.io/style-dictionary/) for design token management.
*   **Icons:** [Lucide](https://lucide.dev/).

**Architecture:**

*   **Entry Point:** `src/routes/+page.svelte` serves as the main entry, setting up context and rendering `src/components/Index.svelte`.
*   **Main Component:** `src/components/Index.svelte` orchestrates the primary sections: Header, Intro, ChartScroll (Scrollytelling), Agent Cards, and Leaderboard.
*   **Data Pipeline:** content is defined in Google Docs/Sheets, configured in `google.config.js`, and fetched into `src/data/` using `npm run gdoc`.
*   **State Management:** Svelte stores in `src/stores/` handle app-wide state like scroll position (`scrollY`), viewport dimensions (`viewport`), and custom logic (`misc.js`).

# Building and Running

**Prerequisites:**

*   Node.js (v18.12.0 or higher recommended).
*   `npm` or `pnpm`.

**Key Commands:**

*   **Install Dependencies:**
    ```bash
    npm install
    # or
    pnpm install
    ```

*   **Start Development Server:**
    ```bash
    npm run dev
    ```
    Runs the app locally (usually at `http://localhost:5173`).

*   **Build for Production:**
    ```bash
    npm run build
    ```
    Generates a static build in the `build/` directory.

*   **Preview Production Build:**
    ```bash
    npm run preview
    ```

*   **Fetch Content (Google Docs/Sheets):**
    ```bash
    npm run gdoc
    ```
    Updates data files in `src/data/` based on `google.config.js`.

*   **Sync Styles:**
    ```bash
    npm run style
    ```
    Regenerates style definitions from `properties/` into `src/styles/` and `src/data/` using Style Dictionary.

*   **Format/Lint:**
    ```bash
    npm run format
    npm run lint
    ```

**Additional Tasks:**
The `tasks/` directory contains specialized scripts for data processing (e.g., `wine-analysis.js`, `imgDownload.js`). These can be run via `npm run <script-name>` as defined in `package.json`.

# Development Conventions

*   **Directory Structure:**
    *   `src/routes/`: Application pages.
    *   `src/components/`: Reusable Svelte components.
    *   `src/components/helpers/`: Utility components (Slider, Toggle, etc.).
    *   `src/components/layercake/`: Chart components.
    *   `src/data/`: Data files (JSON, CSV).
    *   `src/stores/`: Svelte stores.
    *   `src/utils/`: JavaScript utility functions.
    *   `src/styles/`: Global CSS and font definitions.

*   **Styling:**
    *   Global styles are in `src/styles/app.css`.
    *   Design tokens (colors, fonts) should be managed in `properties/` and synced via `npm run style`.
    *   Component-scoped styles are preferred within `.svelte` files.

*   **Data Integration:**
    *   Use `src/data/copy.json` for text content (managed via Google Docs).
    *   Use `getContext("data")` or imports from `$data/` to access data in components.

*   **Code Style:**
    *   Prettier is used for formatting.
    *   Svelte components typically follow the script -> markup -> style order.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/formula-code)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/formula-code)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
