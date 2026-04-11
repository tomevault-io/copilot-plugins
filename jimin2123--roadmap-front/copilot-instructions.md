## roadmap-front

> This document provides a comprehensive overview of the "RoadMap_front" project to give context for future development and maintenance.

# GEMINI.md: Project "RoadMap_front" Context

This document provides a comprehensive overview of the "RoadMap_front" project to give context for future development and maintenance.

## 1. Project Overview

**Purpose:**
"RoadMap_front" is a frontend application for a career and job support platform. Based on the codebase, its features include:
- User authentication (Login, Signup, Session Management).
- Resume management (creating and editing resumes).
- Personalized job recommendations ("Job Match").
- Competency diagnosis to identify skill gaps.
- Information on youth support policies.
- A main dashboard providing access to all these features.

**Core Technologies:**
- **Framework:** React v19
- **Language:** TypeScript
- **Build Tool:** Vite
- **Routing:** React Router DOM v7
- **State Management:** Redux Toolkit
- **Data Fetching & Caching:** TanStack Query (React Query)
- **API Communication:** Axios
- **Styling:** A mix of global CSS (`.css`) and CSS Modules (`.module.css`).
- **Linting/Formatting:** ESLint and Prettier.

**Architecture:**
The project follows a robust, feature-oriented architecture with a clear separation of concerns, as detailed in the `README.md`:
- `src/pages`: Top-level components for each route.
- `src/components`: Reusable UI components, organized by feature.
- `src/layouts`: Components for the main application layout (e.g., `Header`, `Footer`).
- `src/services`: Centralized API call logic using an `axiosInstance`.
- `src/store`: Global state management using Redux Toolkit, with state divided into slices (`authSlice`, `userSlice`, `resumeSlice`).
- `src/hooks`: Custom hooks for encapsulating business logic and side effects (e.g., `useAuth`, `useInitializeSession`).
- `src/routes`: Centralized route definitions for the entire application.
- `src/types`: TypeScript interfaces and type definitions.

## 2. Building and Running

The project's scripts are defined in `package.json`.

- **To run the development server:**
  ```bash
  npm run dev
  ```
  This starts the Vite dev server, which typically runs on the port specified in the `.env` file (`VITE_APP_SERVER_PORT`).

- **To create a production build:**
  ```bash
  npm run build
  ```
  This command type-checks the code (`tsc -b`) and then bundles the application into the `dist/` directory.

- **To lint the code:**
  ```bash
  npm run lint
  ```
  This runs ESLint to check for code quality and style issues based on the configured rules.

- **To preview the production build locally:**
  ```bash
  npm run preview
  ```

## 3. Development Conventions

- **State Management:** Global state is managed with Redux Toolkit. Asynchronous actions (like API calls) are handled using `createAsyncThunk` and processed in the `extraReducers` of each slice. For local component state or server-side state caching, TanStack Query is also available.

- **API Interaction:** All API interactions are centralized within the `src/services` directory. A shared `axiosInstance` is used to configure base URLs and interceptors. API endpoints are proxied during development using Vite's proxy configuration in `vite.config.ts`.

- **Routing:** All application routes are defined in `src/routes/index.tsx`. The file shows evidence of `ProtectedRoute`s being considered or implemented, which is the standard way to handle authenticated routes.

- **Styling:** The project uses a combination of a global stylesheet (`src/styles/global.css`) for base styles and component-specific styles (both `.css` and `.module.css`). This hybrid approach requires careful management to avoid style conflicts.

- **Type Safety:** TypeScript is used throughout the project. Type definitions for complex objects like API responses and Redux state are centralized in the `src/types` directory.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Jimin2123)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Jimin2123)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
