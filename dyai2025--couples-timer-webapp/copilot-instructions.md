## couples-timer-webapp

> This project is a web application called "Couples Timer". Its purpose is to facilitate the "Zwiegespräch nach Moeller," a structured conversation method for couples. The application acts as an impartial timer, strictly managing speaking turns, phases, and breaks to allow partners to focus on the conversation's content rather than on timing.

# GEMINI.md

## Project Overview

This project is a web application called "Couples Timer". Its purpose is to facilitate the "Zwiegespräch nach Moeller," a structured conversation method for couples. The application acts as an impartial timer, strictly managing speaking turns, phases, and breaks to allow partners to focus on the conversation's content rather than on timing.

The application is a single-page application (SPA) built with a modern frontend stack:

*   **Framework:** React
*   **Language:** TypeScript
*   **Build Tool:** Vite
*   **Styling:** Tailwind CSS
*   **Animations:** Framer Motion
*   **Localization:** i18next
*   **Testing:** Vitest, React Testing Library, and Playwright

The architecture is designed with a clean separation of concerns:
1.  **Domain Layer:** Pure, UI-independent TypeScript for all core logic, including a central `SessionEngine` state machine.
2.  **Services Layer:** Encapsulates browser-specific APIs like the Web Audio API for sound and `localStorage` for persistence.
3.  **View Layer:** React components for the user interface.
4.  **View-State Layer:** React Hooks and Context to connect the domain logic to the UI.

Key features include four distinct session modes (Maintain, Commitment, Listening, Custom), a builder for creating custom sequences, audio cues using singing bowl sounds, and localization for German and English.

## Building and Running

The project is set up using Node.js and a package manager (like `npm` or `pnpm`).

**Development Server:**
To run the app in development mode:
```bash
npm install
npm run dev
```

**Testing:**
The project uses Vitest for unit/component tests and Playwright for end-to-end tests.
```bash
# Run unit and component tests
npm run test

# Run end-to-end tests
npx playwright test
```

**Building for Production:**
To create an optimized production build:
```bash
npm run build
```
The output will be in the `dist` directory, ready for deployment to a static hosting service.

## Development Conventions

*   **Structure:** The codebase is organized by architectural layers (`src/domain`, `src/services`, `src/components`, `src/pages`, `src/viewModel`).
*   **State Management:** Core application state is managed by a central, deterministic `SessionEngine`. React's Context API is used to provide this state to the UI components, avoiding direct state manipulation in the UI layer.
*   **Immutability and Purity:** The domain layer is built with pure functions and data structures where possible, making it highly predictable and testable in isolation from the UI.
*   **Testing:** A strong emphasis is placed on testing. The goal is to have high test coverage (≥80%) for the domain and state machine logic. End-to-end tests are used to validate user flows and success criteria.
*   **Styling:** Utility-first CSS with Tailwind CSS is the standard. A central theme is defined in `tailwind.config.js` to ensure visual consistency.
*   **No Shortcuts:** A key principle of the application is enforcing the conversation structure. This is reflected in the code: there are no UI controls or logic to skip or shorten timed phases.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DYAI2025)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DYAI2025)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
