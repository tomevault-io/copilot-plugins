## caffy

> Caffy is an Astro-based platform dedicated to showcasing Colombian culture and traditions. It is built as a highly performant, SEO-friendly blog using modern web technologies.

# Gemini Project Context: Caffy

## Project Overview
Caffy is an Astro-based platform dedicated to showcasing Colombian culture and traditions. It is built as a highly performant, SEO-friendly blog using modern web technologies.

- **Primary Stack:** [Astro](https://astro.build/) (v5), TypeScript, MDX.
- **Integrations:** `@astrojs/mdx`, `@astrojs/rss`, `@astrojs/sitemap`.
- **Styling:** Vanilla CSS based on Bear Blog, using "Atkinson" font for readability.
- **Project Structure:**
    - `src/pages/`: File-based routing for the site.
    - `src/content/blog/`: Markdown and MDX blog posts.
    - `src/components/`: Reusable Astro components.
    - `src/layouts/`: Base layouts for pages and posts.
    - `.specify/`: Orchestration and development lifecycle (Speckit).

## Building and Running
Always use `pnpm` as the package manager for this project.

| Command | Description |
| :--- | :--- |
| `pnpm install` | Install dependencies |
| `pnpm dev` | Start local development server at `localhost:4321` |
| `pnpm build` | Build the production site to `./dist/` |
| `pnpm preview` | Preview the local production build |
| `pnpm astro ...` | Execute Astro CLI commands |

## Development Conventions & Workflows

### Speckit Orchestration
This project employs **Speckit**, a custom agent-driven development framework.
- **Constitution:** Found in `.specify/memory/constitution.md`. All major changes should align with the core principles defined there.
- **Lifecycle:** Development follows a Research -> Design (Contracts/Data Model) -> Task -> Implementation cycle.
- **Commands:** Custom agent commands are available in `.gemini/commands/` (e.g., `speckit.plan`, `speckit.tasks`).
- **Scripts:** Automation scripts are located in `.specify/scripts/powershell/`.

### Coding Standards
- **Styling:** Prefer Vanilla CSS in `src/styles/global.css` or scoped `<style>` tags in Astro components. Avoid TailwindCSS unless explicitly requested.
- **Commits:** Use **Conventional Commits** (e.g., `feat:`, `fix:`, `docs:`, `chore:`).
- **Units:** The project (via Furniture Core memories) may use **METERS** as a native unit for specific logic, though primarily relevant for visual/structural components.
- **Content:** Blog posts require a defined schema in `src/content.config.ts`.

### Testing & Validation
- Validate changes by running `pnpm build` to ensure no TypeScript or Astro compilation errors.
- Check accessibility and SEO patterns (canonical URLs, OpenGraph tags) when adding new pages.

## Key Files
- `astro.config.mjs`: Main Astro configuration.
- `src/consts.ts`: Global site metadata and constants (Site Title, Version, etc.).
- `package.json`: Dependency and script definitions.
- `src/styles/global.css`: Base visual design system.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MrNizzy)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MrNizzy)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
