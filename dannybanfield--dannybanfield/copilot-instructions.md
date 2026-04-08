## dannybanfield

> This file provides project-specific context and guidelines for Gemini CLI when working in this repository.

# GEMINI.md

This file provides project-specific context and guidelines for Gemini CLI when working in this repository.

## Project Overview
**Danny Banfield's Personal Music Artist Website**
A modern, reactive web application built with **SolidJS** to showcase the music career of Danny Banfield. The site features an artist portfolio, media galleries (audio/video), event calendars, and booking information.

### Tech Stack
- **Framework:** [SolidJS](https://www.solidjs.com/) (Reactive UI)
- **Build Tool:** [Vite](https://vitejs.dev/)
- **Styling:** [TailwindCSS](https://tailwindcss.com/) with [DaisyUI](https://daisyui.com/) components
- **Routing:** `@solidjs/router`
- **Meta Management:** `@solidjs/meta`
- **Package Manager:** `pnpm`

## Building and Running

### Development
- `pnpm dev` - Starts the development server at `http://localhost:3000`
- `pnpm clean` - Removes the `dist` directory
- `pnpm nuke` - Removes `dist` and `node_modules` for a fresh start

### Production
- `pnpm build` - Installs dependencies and builds the project for production (outputs to `dist/`)
- `pnpm start` - Previews the production build locally

### Deployment
- The site is hosted on **Cloudflare Pages**.
- Automated deployments are triggered on every push to the `main` branch.

## Project Structure
- `src/index.jsx` - Main entry point and routing configuration.
- `src/pages/` - Top-level page components (Home, Media, Contact, 404).
- `src/components/` - Reusable UI components (Navbar, SlideShow, SpotifyTrack, etc.).
- `src/assets/` - Project-specific media assets (images, logos, SVGs).
- `public/` - Static assets served directly (favicons, manifest, videos).

## Development Conventions

### Architecture & Style
- **SolidJS Patterns:** Use fine-grained reactivity. Prefer functional components and Solid-specific hooks/primitives.
- **Routing:** Client-side routing is handled in `src/index.jsx`.
- **Imports:** Use the `~` alias to reference the `src/` directory (e.g., `import { Navbar } from "~/components/Navbar"`).
- **Styling:** Use TailwindCSS utility classes and DaisyUI component classes. Themes are managed via the `data-theme` attribute (primarily `dark`).
- **Assets:** Import images and media files as ES modules within components for optimized Vite processing.

### Testing & Validation
- **Current Status:** No automated test suite (Vitest/Playwright) is currently configured.
- **Validation:** Manual verification via `pnpm dev` and checking for build errors with `pnpm build`.

### Metadata
- Use `@solidjs/meta` (`Title`, `Meta` tags) within page components to ensure proper SEO and social sharing information.

## Key Features & Integrations
- **Social Media:** Links to Instagram, Facebook, YouTube, and SoundCloud.
- **Embedded Content:** Support for Spotify tracks and Elfsight widgets (Calendar).
- **Interactive UI:** Smooth scrolling, back-to-top functionality, and responsive layouts.
- **Contact Forms:** Specialized sections for song requests and booking inquiries.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dannybanfield)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dannybanfield)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
