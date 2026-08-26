## meal-prep

> Welcome to the Meal Prep codebase. If you are an AI Agent booting up into this repository, this document serves to rapidly onboard you to the existing architectural decisions, edge cases, and design paradigms of this project so you don't have to relearn everything from scratch.

# AI Agent Architecture & Context Document

Welcome to the Meal Prep codebase. If you are an AI Agent booting up into this repository, this document serves to rapidly onboard you to the existing architectural decisions, edge cases, and design paradigms of this project so you don't have to relearn everything from scratch.

## 1. Tech Stack & Tooling Quirks

- **Vite + React + TS**: We are using Vite. The config is in `vite.config.ts`.
- **Firebase**: We use Firebase v10+ modular SDK. The initialization logic is in `src/lib/firebase.ts`.
  - **Auth**: We use Google Sign-In (`signInWithPopup`).
  - **Firestore**: Data is scoped to the user using the path: `artifacts/{appId}/users/{userId}/recipes/{recipeId}`.
- **Linting & Formatting**: We specifically migrated _away_ from ESLint and Prettier. We exclusively use **Oxlint** and **Oxfmt**.
  - **CRITICAL**: We have globally disabled semicolons via `.oxfmtrc.json`. Do not introduce semicolons to the codebase.
  - Run `npm run format` and `npm run lint` to verify your changes.
- **Testing**: We use **Vitest** + React Testing Library (`@testing-library/react`). The setup file is `src/setupTests.ts`.

## 2. Core Components & State Flow

- **`App.tsx`**: Orchestrates global state, including a unified loading screen that handles both Firebase Auth resolution and Firestore initial data fetching to prevent UI flashing.
- **`Dashboard.tsx`**: Renders the user's saved recipes.
- **`Generator.tsx`**: The core AI engine UI.
  - **JSON Parsing**: The Gemini model occasionally returns markdown blockquotes or trails off with extra `}` characters. `Generator.tsx` contains a robust `while` loop that iteratively strips trailing characters until `JSON.parse()` succeeds. Do not remove this failsafe logic.
  - **Prompt Engineering**: The prompt strictly enforces distinct proteins, global cuisines, and common grocery store ingredients. It actively injects the user's high-rated and low-rated past meals to prevent repetition.
  - **Persistence**: Generated options are temporarily saved to `localStorage` so they survive page refreshes until the user makes a selection or discards them.
- **`RecipeDetail.tsx`**: Uses `useParams` to grab the current recipe ID from the URL (`react-router-dom`), then live-binds to the Firestore document. It manages a state-machine for Shopping List, Procedure, and Review tabs.

## 3. UI/UX Design System

- **Styling**: We use Tailwind CSS.
- **Theme**: We use a sleek, dark slate and teal aesthetic. The `index.html` body is hardcoded to `bg-slate-950` to completely eliminate "white flashes" before React mounts.
- **Animations**: We heavily use Tailwind's `animate-in fade-in` and `slide-in` utilities. Any new UI elements must feel dynamic, responsive, and premium. Never build generic, blocky interfaces without hover states, transitions, or rounded corners.

## 4. Workflows for Future Agents

When tasked with expanding this application:

1. Try to test your components directly using `npm run test`.
2. Do not reinstall ESLint. Stick to Oxlint.
3. If modifying the AI generation schema in `Generator.tsx` (now extracted to `src/lib/prompt.ts`), make sure you update the TS interfaces in `src/types/index.ts` and the UI mapping in the detail tabs.

## 5. Agent Operating Preferences

To ensure consistency across chat sessions, adhere to the following critical learnings:

- **Component Architecture**: We strictly avoid massive "facade" hooks or monolithic orchestrators. Push logic (state, API calls, Firestore updates) down directly into the specific leaf components that own it (e.g., `GeneratorForm` handles inputs, `ReviewTab` handles saving reviews).
- **Git Commit Messages**: All commit messages MUST be highly descriptive. They should explain *why* the change happened and provide a small summary of the changes made, while following Git best practices (a short subject line under 50 chars, a blank line, and a detailed body wrapped at 72 chars).

---
> Source: [HauptmannEck/meal-prep](https://github.com/HauptmannEck/meal-prep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-25 -->
