## subforge

> SubForge is a serverless, interactive AI image customizer focused on gay BDSM/kink themes. It allows users to generate high-quality images based on a "Submission Level" scale (1-10).

# GEMINI.md - Project Context: SubForge

## Project Overview
SubForge is a serverless, interactive AI image customizer focused on gay BDSM/kink themes. It allows users to generate high-quality images based on a "Submission Level" scale (1-10).

### Main Technologies
- **Framework:** [Next.js](https://nextjs.org/) (Version 15+ / 16.x) with App Router.
- **Language:** TypeScript.
- **UI/Styling:** React 19, [Tailwind CSS](https://tailwindcss.com/) with a neon/cyberpunk aesthetic.
- **AI Generation (Client-side):** [Puter.js](https://puter.com/) SDK (utilizing `window.puter.ai.txt2img`) with Stable Diffusion 3 (SD3) and SDXL models.
- **AI Generation (Server-side/Fallback):** [fal.ai](https://fal.ai/) using the `fal-ai/flux-pro/v1.1` model via a Next.js API route.
- **Deployment:** Vercel Edge.

### Core Architecture
- **Hybrid Generation:** The application prioritizes client-side generation via Puter.js (which requires no API key when running in the Puter environment). If Puter is unavailable, it falls back to a server-side route (`/api/generate`) powered by fal.ai.
- **Dynamic Prompts:** Image prompts are constructed based on a base prompt plus additional modifiers that scale with the "Submission Level".
- **State Management:** Uses React `useState`, `useEffect`, and `useCallback` for managing generation state, image URLs, and UI progress (Power Exchange Meter).

---

## Building and Running

### Prerequisites
- Node.js (v18+ recommended)
- `FAL_KEY` environment variable (for server-side fallback)

### Commands
- **Development:** `npm run dev` - Starts the Next.js development server.
- **Build:** `npm run build` - Creates an optimized production build.
- **Start:** `npm run start` - Runs the production server.
- **Lint:** `npm run lint` - Executes ESLint checks.
- **Test:** `npm run test` - Runs the Vitest test suite.

---

## Development Conventions

### Code Style & Structure
- **App Router:** All routes are defined in the `app/` directory.
- **API Routes:** Server-side logic resides in `app/api/`.
- **Shared Logic:** Utility functions and shared constants (like prompts and model IDs) are in `lib/`.
- **Types:** Custom type definitions are in `types/` (e.g., `puter.d.ts`).
- **Styling:** Use Tailwind utility classes. For complex neon effects, look at `app/globals.css` and inline styles in `app/page.tsx`.

### AI Integration
- When adding new AI features, consider if they can be implemented client-side via Puter.js first to minimize server costs and API key management.
- Prompts should be kept consistent between `lib/puter-generate.ts` and `app/api/generate/route.ts`.

### Testing
- **Framework:** [Vitest](https://vitest.dev/) with `jsdom` environment.
- **Patterns:**
  - Mock global objects like `window.puter` for testing client-side integrations.
  - Test prompt generation logic (`buildPrompt`) extensively for different level inputs.
  - Tests are located in the `__tests__/` directory.

---

## Key Files
- `app/page.tsx`: The main UI and frontend logic.
- `app/api/generate/route.ts`: Server-side image generation fallback.
- `lib/puter-generate.ts`: Core prompt building and model registry.
- `types/puter.d.ts`: TypeScript definitions for the Puter.js SDK.
- `vitest.config.ts`: Test environment configuration.
- `__tests__/puter-image-generation.test.ts`: Integration tests for image generation logic.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kwizzlesurp10-ctrl) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
