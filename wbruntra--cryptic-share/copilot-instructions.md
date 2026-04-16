## cryptic-share

> Cryptic Share is a full-stack web application for playing and collaborating on cryptic crosswords. It features a modern, responsive frontend and a robust backend that handles real-time collaboration, session management, and AI-powered clue explanations.

# Cryptic Share Crossword Project

## Project Overview

Cryptic Share is a full-stack web application for playing and collaborating on cryptic crosswords. It features a modern, responsive frontend and a robust backend that handles real-time collaboration, session management, and AI-powered clue explanations.

**Key Technologies:**

*   **Runtime:** [Bun](https://bun.com) (used for both frontend and backend tooling)
*   **Frontend:** React, TypeScript, Vite, Tailwind CSS
*   **Backend:** Bun.serve, TypeScript, SSE, SQLite, Knex.js
*   **AI Integration:** OpenAI/Anthropic/Google Gemini (for clue explanations)

## Directory Structure

The project is organized as a monorepo:

*   **`crossword-frontend/`**: The React application source code.
*   **`crossword-backend/`**: The Node.js/Express application source code.
*   **`common/`**: Shared utilities and types (if applicable).
*   **`docs/`**: Project documentation.
*   **`planning/`**: Design documents and feature planning.

## Building and Running

This project exclusively uses **Bun** as the package manager and runtime. Ensure Bun is installed (`curl -fsSL https://bun.sh/install | bash`).

### Frontend (`crossword-frontend/`)

| Command | Description |
| :--- | :--- |
| `bun run dev` | Starts the development server. |
| `bun run build` | Builds the application for production. |
| `bun run lint` | Runs the linter. |
| `bun run preview` | Previews the production build locally. |

### Backend (`crossword-backend/`)

| Command | Description |
| :--- | :--- |
| `bun run dev` | Starts the backend server in development mode (with watch). |
| `bun start` | Starts the backend server in production mode. |
| `bun run migrate:latest` | Runs pending database migrations. |
| `bun run migrate:make <name>` | Creates a new migration file. |
| `bun run create-user` | Helper script to create a new user. |
| `bun run batch-explanations` | Interactive tool for managing AI clue explanations. |

## Development Conventions

### General

*   **Package Manager:** Always use `bun` (`bun install`, `bun add`, `bun run`).
*   **TypeScript:** strict mode is enabled. Ensure all types are properly defined.

### Frontend

*   **Components:** Use functional components with Hooks.
*   **Styling:** Use Tailwind CSS utility classes. Avoid custom CSS files where possible.
*   **State Management:** React Context and Hooks are used for state management.
*   **Networking:** `axios` is used for HTTP requests; SSE is used for real-time updates.

### Backend

*   **Architecture:** Express.js with a service-layer architecture (`routes/` -> `services/` -> `db/`).
*   **Database:** SQLite is the database engine, managed via Knex.js Query Builder.
*   **Migrations:** Database schema changes are handled via Knex migrations (`crossword-backend/migrations/`).
*   **Testing:** Uses Bun's built-in test runner (`bun test`).

## Key Configuration Files

*   **`crossword-frontend/vite.config.ts`**: Vite configuration.
*   **`crossword-frontend/tailwind.config.js`**: Tailwind CSS configuration (inferred).
*   **`crossword-backend/knexfile.ts`**: Database connection and migration configuration.
*   **`crossword-backend/system/crossword.conf`**: Nginx/System configuration (likely).
*   **`AGENTS.md`**: Detailed guidelines for AI agents working on this project.

## Database Schema

The database relies on SQLite. Key tables include:

*   **`users`**: User accounts.
*   **`puzzles`**: Crossword puzzle definitions (grid, clues).
*   **`puzzle_sessions`**: Active game sessions linking users to puzzles.
*   **`clue_explanations`**: Cached AI explanations for clues.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wbruntra) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
