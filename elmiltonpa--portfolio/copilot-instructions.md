## portfolio

> This project is a personal portfolio website that also includes a dedicated interactive games section. It is built using modern web technologies focused on performance and developer experience.

# Gemini Context: Portfolio & Games Project

This project is a personal portfolio website that also includes a dedicated interactive games section. It is built using modern web technologies focused on performance and developer experience.

## Project Overview

*   **Purpose:** A professional portfolio to showcase personal information and projects, with an added fun element of mini-games (Sudoku, Tic-Tac-Toe, etc.).
*   **Main Technologies:**
    *   **Framework:** [Astro](https://astro.build/) (v5+) - A modern web framework for content-driven websites.
    *   **Styling:** [Tailwind CSS](https://tailwindcss.com/) (v4) - A utility-first CSS framework, integrated via Vite.
    *   **Runtime/Package Manager:** [Bun](https://bun.sh/) - Used for fast dependency management and execution.
*   **Architecture:**
    *   **Routing:** File-based routing handled by `src/pages/`.
    *   **Components:** Organized by domain in `src/components/` (e.g., `Home/`, `Games/`).
    *   **Sections:** High-level global components like `Header.astro` and `Footer.astro` reside in `src/sections/`.
    *   **Layouts:** Reusable page wrappers found in `src/layouts/`.

## Building and Running

Commands should be executed using `bun` from the project root:

| Command | Action |
| :--- | :--- |
| `bun install` | Installs project dependencies. |
| `bun dev` | Starts the local development server at `http://localhost:4321`. |
| `bun build` | Builds the static site for production in the `./dist/` directory. |
| `bun preview` | Locally previews the production build. |
| `bun astro ...` | Executes Astro CLI commands (e.g., `bun astro check`). |

## Development Conventions

*   **Component Structure:** Favor small, reusable components. Components specific to a page should be grouped in a subfolder within `src/components/` (e.g., `src/components/Home/About.astro`).
*   **Styling:** Use Tailwind CSS utility classes directly in Astro components. Global styles and `@theme` variables should be defined in `src/styles/global.css`.
*   **Game Organization:** Each game should have its own subfolder in `src/components/Games/` to encapsulate its specific logic and sub-components (e.g., `src/components/Games/Sudoku/Grid.astro`).
*   **TypeScript:** The project is configured with TypeScript (see `tsconfig.json`). Ensure type safety in scripts where possible.
*   **File Naming:** Use PascalCase for Astro components (e.g., `HeroSection.astro`) and camelCase or kebab-case for utility scripts.

## Current Progress / TODO

- [ ] Implement `Hero`, `About`, `Projects`, and `Contact` components in `src/components/Home/`.
- [ ] Design and implement the Games landing page in `src/pages/games/index.astro`.
- [ ] Develop individual games (e.g., Tic-Tac-Toe, Sudoku) as components.
- [ ] Add individual routes for each game in `src/pages/games/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/elmiltonpa)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/elmiltonpa)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
