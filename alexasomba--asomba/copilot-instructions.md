## asomba

> This document provides context and guidelines for interacting with the Alex Asomba Portfolio codebase.

# GEMINI.md - Alex Asomba Portfolio

This document provides context and guidelines for interacting with the Alex Asomba Portfolio codebase.

## Project Overview

A modern, responsive, and multilingual portfolio website for Alex Asomba, a Fullstack AI Engineer and Computer Scientist.

- **Framework:** [Astro 5](https://astro.build/)
- **Styling:** [Tailwind CSS 4](https://tailwindcss.com/)
- **Language:** TypeScript
- **Output:** Static site generation

### Key Features
- **Multilingual Support (i18n):** Supports 5 languages with seamless switching via a dropdown:
  - English (EN - Default)
  - French (FR)
  - Portuguese (PT)
  - Nigerian Pidgin (WAZ)
  - Chinese (ZH)
- **Dark Mode:** System-aware theme toggle with persistence in `localStorage` and cookies.
- **Modern UI:** Responsive design, animated borders ("rainbow animations"), and high-performance static rendering.

---

## Building and Running

The project uses `pnpm` as the package manager.

- **Install Dependencies:** `pnpm install`
- **Development Server:** `pnpm dev` (Runs at [http://localhost:4321](http://localhost:4321))
- **Production Build:** `pnpm build` (Generates files in `dist/`)
- **Preview Build:** `pnpm preview`
- **Check Types/Code:** `pnpm astro check`

---

## Project Structure

- `src/assets/`: Static assets like images and SVG icons.
- `src/components/`: Reusable Astro components (e.g., `Header.astro`, `Footer.astro`).
- `src/layouts/`: Base layout (`Layout.astro`) containing global styles and scripts.
- `src/pages/`: Main application pages and routes.
  - `index.astro`: Home page with profile and tech stack.
  - `work.astro`: Professional experience timeline.
  - `projects.astro`: Portfolio showcase.
  - `contact.astro`: Contact information.
- `src/project-descriptions/`: Markdown files for detailed project descriptions. Primarily supports EN and ES (legacy); others fall back to EN.
- `src/styles/`: Global CSS and Tailwind directives.

---

## Development Conventions

### Multilingual Support (i18n)
The project uses a custom attribute-based i18n system managed in `src/layouts/Layout.astro`.
- **Implementation:** Use `data-{lang}` attributes on HTML elements (e.g., `data-en`, `data-fr`, etc.).
- **Example:**
  ```html
  <p data-en="Hello" data-fr="Bonjour">Hello</p>
  ```
- **Placeholders/Labels:** Use `data-{lang}-placeholder` and `data-{lang}-label`.
- **Logic:** The `window.updatePageContent()` function in the layout handles the DOM updates. Users can select languages directly via a dropdown in the header, which triggers the `window.setLang(lang)` function.

### Theme Management
- Dark mode is implemented using Tailwind's `dark:` variant and a `.dark` class on the `<html>` element.
- Theme state is persisted in `localStorage` ('theme') and a cookie ('theme').

### Styling & Design
- **Tailwind CSS 4:** Uses the new `@tailwindcss/vite` integration.
- **Animations:** Custom CSS animations (e.g., `animated-border`, `fade-in`) are defined in `src/styles/global.css`.
- **Visibility:** Use `.locale-{lang}` classes for language-specific visibility management.

---

## Deployment
The project is optimized for deployment on Vercel or Netlify. The `astro.config.mjs` is configured for static output.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alexasomba) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
