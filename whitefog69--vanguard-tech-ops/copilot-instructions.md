## vanguard-tech-ops

> This workspace contains the source code and assets for the "Vanguard Tech Ops" professional corporate website. The project is a modern, high-end React 19 application focusing on technical authority, digital sovereignty, and "Quiet Luxury" aesthetics.

# Vanguard Tech Ops - Project Context

This workspace contains the source code and assets for the "Vanguard Tech Ops" professional corporate website. The project is a modern, high-end React 19 application focusing on technical authority, digital sovereignty, and "Quiet Luxury" aesthetics.

## Project Structure

- `Website/`: The main React application directory.
  - `src/`: Source code for the React application.
    - `components/`: Reusable UI components (e.g., Layout, Navbar, Footer).
    - `pages/`: Top-level route components (Home, Services, Contact, etc.).
    - `pages/services/`: Specialized service pages (Cloud, AISystems, WebDev, Shopify).
    - `lib/`: Utility functions (e.g., `cn` for class merging).
    - `App.tsx`: Routing configuration using React Router 7.
    - `index.css`: Global styles and Tailwind 4 theme configuration.
  - `public/`: Static assets for the website.
- `*.mp4`, `*.png`: Source media assets (located in the root directory for reference/source).

## Core Technologies

- **Framework**: React 19
- **Build Tool**: Vite 6
- **Language**: TypeScript
- **Routing**: React Router 7
- **Styling**: Tailwind CSS 4
- **Animations**: Framer Motion (`motion/react`)
- **Icons**: Lucide React
- **AI Integration**: `@google/genai` (for Gemini API)

## Building and Running

All commands should be run from within the `Website/` directory.

- `npm install`: Install dependencies.
- `npm run dev`: Start development server on `http://localhost:3000`.
- `npm run build`: Create a production build in `dist/`.
- `npm run lint`: Run TypeScript type checking (`tsc --noEmit`).
- `npm run preview`: Locally preview the production build.
- `npm run clean`: Remove the `dist/` directory.

## Development Conventions

- **Styling**: Uses Tailwind 4 with a custom theme adhering to **Material Design 3** naming conventions (e.g., `primary-container`, `on-surface`, `surface-container-low`).
- **Aesthetic**: "Quiet Luxury" - glassmorphism, glowing cobalt/violet accents, smooth transitions, and technical authority.
- **Class Merging**: Always use the `cn` utility from `src/lib/utils.ts` for dynamic Tailwind classes.
- **Animations**: Prefer `motion` components from `motion/react` for entrance and hover effects.
- **Type Safety**: Strict TypeScript usage for all components and utilities.
- **Environment**: Requires `GEMINI_API_KEY` in `.env.local` for AI features.

## Key Files

- `Website/src/App.tsx`: Main routing and application shell.
- `Website/src/index.css`: Tailwind 4 theme and global utility classes (e.g., `.glass-card`).
- `Website/src/components/Layout.tsx`: Shared layout, Navbar with services dropdown, and Footer.
- `Website/package.json`: Project dependencies and scripts.
- `Website/vite.config.ts`: Vite configuration including Tailwind integration.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/whitefog69)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/whitefog69)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
