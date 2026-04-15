## nova-tech-main

> **0422 Nova Tech** is a modern, single-page React application built with Vite. It serves as a portfolio/landing page for a software engineering and industrial design firm. The application features a distinctive "industrial/high-tech" aesthetic, utilizing dark modes, monospaced fonts, and glitch effects.

# Nova Tech - Project Context

## Project Overview

**0422 Nova Tech** is a modern, single-page React application built with Vite. It serves as a portfolio/landing page for a software engineering and industrial design firm. The application features a distinctive "industrial/high-tech" aesthetic, utilizing dark modes, monospaced fonts, and glitch effects.

**Key Technologies:**
- **Framework:** React 19
- **Build Tool:** Vite 6
- **Language:** TypeScript
- **Styling:** Tailwind CSS
- **Icons:** Lucide React

## Architecture & Structure

The project follows a standard Vite + React structure:

- **`App.tsx`**: The main entry point composed of stacked sections (`Hero`, `Origin`, `EngineRoom`, etc.).
- **`components/`**: Contains individual UI sections and reusable components.
    - `Hero.tsx`: Landing section with animated logo and CTA.
    - `Contact.tsx`, `Portfolio.tsx`, etc.: Other page sections.
- **`types.ts`**: Centralized TypeScript interfaces for data models (e.g., `ServiceItem`, `PortfolioItem`).
- **`Nova-Tech-main/`**: The root directory for the source code.

## Building and Running

### Prerequisites
- Node.js (v18+ recommended)
- npm

### Key Commands

Run these commands from the `Nova-Tech-main` directory:

| Command | Description |
| :--- | :--- |
| `npm install` | Install project dependencies. |
| `npm run dev` | Start the local development server (uses Vite). |
| `npm run build` | Build the application for production. |
| `npm run preview` | Preview the production build locally. |

### Environment Variables
- Create a `.env.local` file in the project root if integrating with Gemini API.
- Required Variable: `GEMINI_API_KEY` (as noted in README, though current code inspection shows primarily UI focus).

## Development Conventions

- **Component Style:** Functional components using `React.FC` or direct function declarations.
- **Styling:** extensive use of Tailwind CSS utility classes (e.g., `bg-[#0a0a0a]`, `font-display`).
- **Responsiveness:** Mobile-first approach with `md:` and `lg:` breakpoints.
- **Type Safety:** Strict use of TypeScript interfaces defined in `types.ts`.
- **Assets:** Images and icons are either imported components or referenced via URL/paths.

## Task Guide for AI Agents

- **Modifying UI:** Check `App.tsx` to see the order of sections, then modify the specific file in `components/`.
- **Adding Data:** Update `types.ts` if the data structure changes, then update the relevant component.
- **Styling Changes:** Use Tailwind classes. Look for custom font classes like `font-display` and `font-mono`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lindokuhlethebe-commits)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lindokuhlethebe-commits)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
