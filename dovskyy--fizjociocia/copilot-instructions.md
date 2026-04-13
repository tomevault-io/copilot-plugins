## fizjociocia

> This file provides foundational mandates and context for Gemini CLI when working in the **Fizjociocia** project.

# GEMINI.md

This file provides foundational mandates and context for Gemini CLI when working in the **Fizjociocia** project.

## Project Overview

**Fizjociocia** is a modern, single-page landing site for a pediatric physiotherapist, **Marcelina Kochanowska**. The site serves as a professional portfolio and service showcase, focusing on NDT-Bobath and Sensory Integration (SI) therapies.

- **Main Technologies**: React 19, TypeScript, Vite.
- **Styling Strategy**: Utility-first CSS using **Tailwind CSS via CDN**.
- **Visual Identity**: Pastel colors (pink, mint, yellow, blue), organic shapes ("scribble-borders"), and friendly typography (Playfair Display, Plus Jakarta Sans, Kalam).
- **Architecture**: A single-page application (SPA) composed of modular sections (`Navbar`, `Hero`, `About`, `Patients`, `Services`, `VisitProcess`, `Equipment`, `Stories`, `Contact`, `Footer`).

## Building and Running

| Command | Description |
|---|---|
| `npm install` | Install project dependencies. |
| `npm run dev` | Start the development server (default: `http://localhost:3000`). |
| `npm run build` | Build the application for production. |
| `npm run preview` | Preview the production build locally. |

## Development Conventions

### Styling (Critical Mandate)
- **Tailwind via CDN**: Tailwind is loaded via a `<script>` tag in `index.html`. There is **no** `tailwind.config.js`.
- **Configuration**: Theme extensions (colors, fonts, animations) are defined in a `<script>` block within `index.html`.
- **Custom Utilities**: Custom CSS classes like `scribble-border`, `floating-slow`, and `paper-texture` are defined in a `<style type="text/tailwindcss">` block in `index.html`.
- **Colors**:
    - `primary`: `#ffcccb` (Pastel Pink)
    - `pastel-mint`: `#b9f2e1`
    - `pastel-yellow`: `#fef3c7`
    - `pastel-blue`: `#bae6fd`
    - `soft-beige`: `#fdfaf6` (Main Background)

### Components
- **Functional Components**: All components are React functional components using TypeScript (`React.FC`).
- **Path Aliases**: Use `@/` to reference the project root (e.g., `import Navbar from '@/components/Navbar'`).
- **Location**: All section components are located in the `components/` directory.

### Assets
- **Images**: Served from the `/assets/` directory. Prefer `.webp` format for performance.
- **Fonts**: Integrated via Google Fonts (Playfair Display, Plus Jakarta Sans, Kalam).

### Content
- **Language**: Primary language is Polish (`pl`).
- **Tone**: Professional, warm, and child-friendly.

## Key Files
- `App.tsx`: The main application layout and section orchestrator.
- `index.html`: The core HTML template containing Tailwind configuration and global styles.
- `vite.config.ts`: Vite configuration, including port settings and environment variable definitions.
- `assets/fizjociocia-content.md`: Contains raw copy and content for the website.

## Environment Variables
- `GEMINI_API_KEY`: Loaded from `.env.local` and exposed as `process.env.GEMINI_API_KEY` (and `process.env.API_KEY`) for use in the application.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dovskyy)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dovskyy)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
