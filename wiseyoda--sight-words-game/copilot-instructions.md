## sight-words-game

> **Sight Words Game** is a game designed for kindergarten children to learn sight words. This repository contains the source code and documentation for the project.

# GEMINI.md

## Project Overview
**Sight Words Game** is a game designed for kindergarten children to learn sight words. This repository contains the source code and documentation for the project.

## Project Status
**New/Initializing:** The project is currently in the early setup phase. The repository contains governance and guideline files (`AGENTS.md`, `CLAUDE.md`, `README.md`) but the application code has not yet been scaffolded.

## Intended Technology Stack
Based on the repository guidelines (`AGENTS.md`), the project aims to use:
*   **Runtime/Package Manager:** Node.js and npm
*   **Language:** TypeScript
*   **Frontend Framework:** React (likely with Vite)
*   **Testing:** React Testing Library

## Intended Architecture & Structure
Future development should adhere to the following structure:
*   `src/` - Gameplay logic and UI
    *   `src/components/` - Reusable UI views
    *   `src/game/` - Game state and logic
*   `public/` - Static entry HTML
*   `assets/` - Visual and audio assets
*   `tests/` - Automated tests (or co-located `*.test.tsx`)
*   `scripts/` - Helper scripts

## Development Conventions

### Commands (Planned)
*   `npm install` - Install dependencies
*   `npm run dev` - Start local development server
*   `npm run build` - Build for production
*   `npm run preview` - Preview production build
*   `npm test` - Run automated tests
*   `npm run lint` - Run linting

### Coding Style
*   Use **TypeScript** for UI and game logic.
*   **Naming:** `PascalCase` for components, `usePrefix` for hooks, `camelCase` for utilities.
*   **Formatting:** 2-space indentation, semicolons, consistent imports.
*   **Commits:** Follow Conventional Commits (`feat:`, `fix:`, `chore:`, `docs:`).

### Accessibility & Safety
*   Design for high contrast and large tap targets (kindergarten accessible).
*   Avoid rapid flashing/distracting loops.
*   **Privacy:** Do not collect personal data; keep state local.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wiseyoda)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/wiseyoda)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
