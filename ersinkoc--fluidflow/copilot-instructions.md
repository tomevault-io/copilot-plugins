## fluidflow

> **FluidFlow** is a "Sketch-to-App" AI prototyping tool that transforms wireframes and sketches into functional React applications. It leverages multiple AI providers (Gemini, OpenAI, Anthropic, etc.) to generate code, manage projects, and provide a live preview environment.

# FluidFlow - Gemini Context

## Project Overview
**FluidFlow** is a "Sketch-to-App" AI prototyping tool that transforms wireframes and sketches into functional React applications. It leverages multiple AI providers (Gemini, OpenAI, Anthropic, etc.) to generate code, manage projects, and provide a live preview environment.

## Tech Stack
-   **Frontend:** React 19, TypeScript 5.9, Vite 7, Tailwind CSS 4, Monaco Editor.
-   **Backend:** Node.js, Express 5, `tsx` for execution.
-   **AI Integration:** Multi-provider architecture (Gemini, OpenAI, Anthropic, Z.AI, Ollama, OpenRouter).
-   **State Management:** React Context (`AppContext`), specialized hooks (`useProject`, `useCodeGeneration`).
-   **Testing:** Vitest, JSDOM.
-   **Build/Dev:** Vite (Frontend), `tsx` (Backend).

## Architecture

### Core Components
-   **ControlPanel (`components/ControlPanel/`)**: The "Brain" of the UI. Handles user input (chat/images), orchestrates AI calls, and manages settings.
-   **PreviewPanel (`components/PreviewPanel/`)**: The "View". Displays the generated app via an iframe, handles the code editor (Monaco), and shows console/network logs.
-   **Server (`server/`)**: Express API handling project persistence, git operations, and local file management.
-   **Services (`services/`)**:
    -   `ai/`: Abstracted AI provider logic.
    -   `conversationContext.ts`: Manages token limits and auto-compaction of chat history.
    -   `wipStorage.ts`: IndexedDB wrapper for persisting uncommitted changes across reloads.
-   **Virtual File System**: Projects are stored as a `Record<string, string>` object in memory and synced to the `projects/` directory on disk.

### Data Flow
1.  **Input:** User uploads an image or types a prompt in `ControlPanel`.
2.  **Processing:** `ControlPanel` calls `services/ai` to stream a response.
3.  **Parsing:** The response is parsed by `utils/cleanCode.ts` to extract file contents.
4.  **Review:** A `DiffModal` presents changes for user approval.
5.  **Execution:** Upon approval, files are updated in `AppContext` and written to disk via the backend.
6.  **Preview:** `PreviewPanel` updates the iframe to reflect the new code.

## Key Development Commands

| Command | Description |
| :--- | :--- |
| `npm run dev` | Starts **both** Frontend (port 3100) and Backend (port 3200). |
| `npm run dev:frontend` | Starts only the Vite frontend dev server (HTTPS enabled). |
| `npm run dev:server` | Starts only the Express backend server. |
| `npm test` | Runs tests in watch mode (Vitest). |
| `npm run type-check` | Runs TypeScript compiler check (`tsc --noEmit`). |
| `npm run lint` | Runs ESLint (Zero tolerance for warnings). |
| `npm run lint:fix` | Auto-fixes ESLint issues. |

## Critical Files & Directories

-   **`contexts/AppContext.tsx`**: Central state hub. If you need to access global state (files, current project, git status), look here.
-   **`App.tsx`**: Main application layout and modal manager.
-   **`server/index.ts`**: Backend entry point.
-   **`services/ai/index.ts`**: The factory and manager for AI providers.
-   **`utils/cleanCode.ts`**: Critical logic for parsing AI responses into usable code.
-   **`PLAN.md`**: Contains the roadmap and refactoring plans. Check this before extensive refactoring.
-   **`CLAUDE.md`**: Contains similar context instructions, useful for cross-reference.

## Development Conventions

-   **Type Safety:** Strict TypeScript is enforced. No `any` allowed in source code (relaxed in tests).
-   **Styling:** Tailwind CSS 4 is the standard. Avoid inline styles or separate CSS files where possible.
-   **Imports:** Use path aliases (e.g., `@/components`, `@/utils`) as defined in `vite.config.ts` and `tsconfig.json`.
-   **Testing:** Write unit tests for new utilities and integration tests for critical flows. Use `vitest`.
-   **Security:** Be mindful of XSS and path traversal. Validate all inputs, especially file paths and user-provided content.

## Configuration
-   **Environment Variables:** Stored in `.env.local` (e.g., `GEMINI_API_KEY`, `VITE_API_URL`).
-   **Vite Config:** `vite.config.ts` handles the build and dev server proxy settings.
-   **Port Configuration:** Frontend defaults to `3100`, Backend to `3200`.

---
> Source: [ersinkoc/FluidFlow](https://github.com/ersinkoc/FluidFlow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
