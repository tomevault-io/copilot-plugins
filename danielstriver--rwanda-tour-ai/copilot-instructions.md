## rwanda-tour-ai

> Rwanda Tourist Assistant is a UI-first MVP prototype of an AI-powered tourism recommendation platform for Rwanda. It provides a polished landing experience, preference selection flow, and static recommendation cards, serving as a foundation for future AI integrations.

# Rwanda Tourist Assistant - Project Context

Rwanda Tourist Assistant is a UI-first MVP prototype of an AI-powered tourism recommendation platform for Rwanda. It provides a polished landing experience, preference selection flow, and static recommendation cards, serving as a foundation for future AI integrations.

## Project Overview

- **Purpose:** Provide personalized travel recommendations for Rwanda based on user preferences.
- **Status:** MVP Prototype (UI-focused with static data).
- **Tech Stack:**
  - **Framework:** React 19 (TypeScript)
  - **Build Tool:** Vite
  - **Styling:** Chakra UI (with custom theme and dark mode support)
  - **Animations:** Framer Motion
  - **Icons:** Lucide React
  - **Deployment:** Vercel

## Architecture

The project follows a feature-based structure within the `src` directory:

- `src/components/`: Reusable UI components (Navbar, Footer, Hero, etc.).
- `src/features/preferences/`: Logic and UI for selecting experience types, budget, and duration.
- `src/features/recommendations/`: Logic and UI for displaying destination recommendations.
- `src/hooks/`: Custom React hooks (e.g., `usePreferenceState`, `useAutoSlide`).
- `src/pages/`: Page-level components (currently only `RwandaTourPage.tsx`).
- `src/types/`: TypeScript interfaces and types.
- `src/utils/`: Constants, sample data (`SAMPLE_RECOMMENDATIONS`), and helper functions.

## Development Workflow

### Key Commands

- `npm install`: Install project dependencies.
- `npm run dev`: Start the Vite development server.
- `npm run build`: Compile TypeScript and build the production-ready assets.
- `npm run preview`: Preview the locally built production version.

### Coding Conventions

- **Component Style:** Functional components using TypeScript and Chakra UI.
- **Styling:** Prefer Chakra UI's style props and `semanticTokens` for consistent theme and dark mode support.
- **State Management:** Use custom hooks (e.g., `usePreferenceState`) to encapsulate feature-specific state.
- **Animations:** Use `framer-motion` for section transitions and interactive elements.
- **File Naming:** PascalCase for components (`Navbar.tsx`), camelCase for hooks and utils (`useAutoSlide.ts`).

## Key Files for Context

- `package.json`: Project dependencies and scripts.
- `src/main.tsx`: Entry point and Chakra UI theme configuration.
- `src/pages/RwandaTourPage.tsx`: Main layout and section orchestration.
- `src/utils/constants.ts`: Contains the `SAMPLE_RECOMMENDATIONS` data model.
- `src/types/recommendation.ts`: Core data structures for the recommendation engine.

## Future Roadmap (Context for AI)

- Integration with Vercel Functions for AI-driven recommendations.
- Dynamic data fetching from external APIs or databases.
- Enhanced interactive elements like a voice-enabled travel assistant.
- Expanded localization and multilingual support.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/danielstriver)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/danielstriver)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
