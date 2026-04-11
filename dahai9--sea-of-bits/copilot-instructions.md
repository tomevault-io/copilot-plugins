## sea-of-bits

> **Sea_of_Bits** is a high-performance, minimalist technical blog built with **Astro 6** and **TypeScript**. It serves as a personal space for sharing technical deep dives and creative thoughts.

# GEMINI.md - Project Instructions

## Project Overview
**Sea_of_Bits** is a high-performance, minimalist technical blog built with **Astro 6** and **TypeScript**. It serves as a personal space for sharing technical deep dives and creative thoughts.

### Key Technologies
- **Framework**: Astro (Static Site Generation)
- **Content**: Markdown and MDX (`@astrojs/mdx`)
- **Environment**: Nix Flakes (`flake.nix`) and `direnv`
- **Deployment**: GitHub Pages (Base path: `/Sea_of_Bits`)
- **Styling**: Vanilla CSS for a clean, text-focused minimalist aesthetic.

---

## 🛠 Building and Running

### Development Environment
The project uses **Nix** for a reproducible environment.
- **Enable Environment**: Run `direnv allow` or `nix develop`.
- **Node Version**: Requires Node.js `>=22.12.0`.

### Commands
- **Start Dev Server**: `npm run dev` (Runs Astro in dev mode)
- **Build for Production**: `npm run build` (Generates static files in `dist/`)
- **Preview Build**: `npm run preview` (Serves the `dist/` directory locally)
- **Type Checking**: `npm run typecheck` (Runs `astro check` for content and TypeScript validation)

---

## 📦 Content Structure & Conventions

### Blog Posts (`src/content/blog/`)
All blog posts are stored as `.md` or `.mdx` files.
- **Schema**: Defined in `src/content.config.ts`.
- **Required Frontmatter**:
  - `title`: String (Article title)
  - `description`: String (Short summary)
  - `pubDate`: Date (Publication date, e.g., 'Mar 19 2026')
  - `heroImage`: Image path (Processed via Astro's `image()` helper)
  - `updatedDate`: Optional Date

### Images (`src/assets/`)
Store all article-specific assets (banners, diagrams) here. Use relative paths in Markdown frontmatter to reference them.

### Layouts (`src/layouts/`)
- `BlogPost.astro`: The primary layout for all articles. It uses the `Image` component from `astro:assets` to handle hero images.

---

## 👨‍💻 Development Conventions
1.  **Strict Typing**: Ensure all content matches the defined schema. Use `npm run typecheck` frequently.
2.  **Asset Handling**: Always use the `Image` component from `astro:assets` for optimized image rendering.
3.  **Minimalist Style**: Maintain the clean, text-focused CSS in `src/styles/global.css`.
4.  **Deployment**: Any push to `main` triggers an automatic deployment via GitHub Actions.

---

## 🤖 AI Interaction Guidelines
- When adding new blog posts, ensure the frontmatter is complete and follows the schema.
- If modifying layouts, prioritize accessibility and maintain the minimalist aesthetic.
- Always run `npx astro check` after modifying content or schemas to ensure data integrity.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dahai9)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dahai9)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
