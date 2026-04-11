## kletta

> **Kletta** is a specialized AI Agent Workspace designed to assist users in winning Kaggle competitions. It leverages a team of specialized AI agents (Scout, Researcher, Strategist, Coder, etc.) to analyze competitions, suggest research directions, and manage tasks.

# Kletta

## Project Overview

**Kletta** is a specialized AI Agent Workspace designed to assist users in winning Kaggle competitions. It leverages a team of specialized AI agents (Scout, Researcher, Strategist, Coder, etc.) to analyze competitions, suggest research directions, and manage tasks.

The application is built with **React**, **Vite**, and **TypeScript**. It integrates with multiple AI providers (**Google Gemini**, **OpenRouter**, **OpenAI**) and the **Kaggle API** to provide a comprehensive workspace for data science competitions.

## Architecture & Key Concepts

*   **Agents:** The core logic involves switching between different agent personas (e.g., `@scout`, `@coder`) defined in `services/geminiService.ts`.
*   **Providers:** Supports a fallback mechanism across multiple AI providers. Users can configure API keys in the settings.
*   **Data Persistence:** Uses `services/storageService.ts` (likely IndexedDB wrapper) to persist competitions, messages, and memory state locally.
*   **Kaggle Integration & Proxy:** Connects to Kaggle via a local **Bun proxy** (`server.ts`) to fetch active competitions and search for datasets. This solves browser CORS issues.
*   **UI/UX:** A responsive 3-pane layout (Sidebar Left, Chat Area, Sidebar Right) built with React and styled with Tailwind CSS concepts.

## Key Files & Directories

*   `src/App.tsx`: The main entry point managing global state and layout.
*   `server.ts`: Native Bun server acting as an API proxy for Kaggle and static file server for production.
*   `src/services/geminiService.ts`: Handles AI model interactions and agent switching.
*   `src/services/kaggleService.ts`: Manages interactions with the Kaggle API (via local proxy).
*   `src/components/`: Contains UI components.
*   `types.ts`: Centralized TypeScript definitions.
*   `vite.config.ts`: Configuration for the Vite build tool, includes proxy for development.

## Building and Running

**Prerequisites:** Bun

1.  **Install Dependencies:**
    ```bash
    bun install
    ```

2.  **Environment Setup:**
    Create a `.env.local` file with API keys:
    ```bash
    VITE_GEMINI_API_KEY=your_gemini_key
    VITE_OPENROUTER_API_KEY=your_openrouter_key
    VITE_OPENAI_API_KEY=your_openai_key
    VITE_DEFAULT_PROVIDER=gemini
    ```

3.  **Run Development Server:**
    ```bash
    bun run dev
    ```
    This runs both the Vite frontend and the Bun proxy server concurrently.

4.  **Build for Production:**
    ```bash
    bun run build
    ```

5.  **Run Production Server:**
    ```bash
    bun start
    ```

## Development Conventions

*   **Runtime:** Bun is the preferred runtime for installation, execution, and server-side tasks.
*   **Tech Stack:** React 19, Vite, TypeScript.
*   **Styling:** Utility-first CSS with `clsx` for conditional classes.
*   **State Management:** React `useState` and `useEffect` for local state; `storageService` for persistence.
*   **Type Safety:** Strict TypeScript usage.

---

# Gemini Instructions

Fast Apply: IMPORTANT: Use `edit_file` over `str_replace` or full file writes. It works with partial code snippets—no need for full file content.

Warp Grep: warp-grep is a subagent that takes in a search string and tries to find relevant context. Best practice is to use it at the beginning of codebase explorations to fast track finding relevant files/lines. Do not use it to pin point keywords, but use it for broader semantic queries. "Find the XYZ flow", "How does XYZ work", "Where is XYZ handled?", "Where is <error message> coming from?"

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ash-Blanc)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Ash-Blanc)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
