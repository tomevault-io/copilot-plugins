## speed

> - The project is an Electron + React + Vite + Zustand monolith.

# Speed - LLM Instructions

## Core Principles
- The project is an Electron + React + Vite + Zustand monolith. 
- Write concise, production-ready code.
- Optimize for standard ES6 syntax and arrow functions.

## Technical Constraints
- **State Management**: Use ONLY `zustand` (`src/store/agentStore.ts`). Do NOT use Redux or Context API for state.
- **Styling**: Use TailwindCSS exclusively via utility classes. Do NOT use `@apply` in CSS files unless absolutely necessary.
- **UI Framework**: Use `lucide-react` for icons and `framer-motion` for animations.
- **Routing**: The app uses `react-router-dom` with HashRouter (expected for Electron).

## Orchestration Logic
- The core simulation logic lives inside `agentStore.ts` inside a monolithic `tick()` function state modifier.
- The `useOrchestrator` hook in `src/utils/orchestrator.ts` drives this `tick()` globally at a defined interval within `App.tsx`.
- When adding tasks or agent behaviors, they must be cleanly integrated into the phase-based logic inside `tick()` (`planning` -> `approval` -> `architecture` -> `development`).
- Do not attempt to refactor the `.tick()` function into separate slices unless explicitly instructed by the user, as the monolithic store enables easy cross-entity state modifications per frame.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vinaycookscode) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
