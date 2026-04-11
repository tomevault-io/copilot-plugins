## timeless-library

> This document provides context for the Gemini code assistant to understand the Timeless Library project.

# Gemini Code Assistant Context

This document provides context for the Gemini code assistant to understand the Timeless Library project.

## Project Overview

The Timeless Library is a web application built with [Astro](https://astro.build/) that provides a curated collection of classic books. The goal of the project is to make classic literature more accessible and engaging for contemporary readers by modernizing the language and providing helpful annotations.

The project uses the following technologies:

*   **Framework:** [Astro](https://astro.build/)
*   **Styling:** [Tailwind CSS](https://tailwindcss.com/)
*   **Language:** [TypeScript](https://www.typescriptlang.org/)
*   **Testing:** [Vitest](https://vitest.dev/) and [Jest](https://jestjs.io/)

The application's architecture is based on Astro's component-based structure. The content for the books is managed in Markdown files in the `src/content/books` directory. The schema for the book content is defined in `src/content/config.ts`.

## Building and Running

The following commands are used to build, run, and test the project:

*   **Development:** `npm run dev` - Starts the development server at `http://localhost:4321`.
*   **Build:** `npm run build` - Builds the application for production.
*   **Preview:** `npm run preview` - Previews the production build locally.
*   **Test:** `npm run test` - Runs the test suite using Vitest.
*   **Lint:** `npm run lint` - Lints the TypeScript files using ESLint.
*   **Format:** `npm run format` - Formats the code using Prettier.

## Development Conventions

*   **Coding Style:** The project uses [Prettier](https://prettier.io/) for code formatting and [ESLint](https://eslint.org/) for linting. The configuration for these tools can be found in `.prettierrc` and `.eslintrc.cjs` respectively.
*   **Testing:** The project uses [Vitest](https://vitest.dev/) for unit and component testing. Test files are located in the `__tests__` directory alongside the files they test.
*   **Contribution Guidelines:** Contributions are welcome. Please follow the guidelines in the `README.md` file.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bgreenawald)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bgreenawald)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
