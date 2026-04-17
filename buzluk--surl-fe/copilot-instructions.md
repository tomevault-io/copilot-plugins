## surl-fe

> This is a frontend application for a URL shortener service, built with **React** and **TypeScript**. It uses **Vite** for fast development and bundling. The application allows users to sign up, sign in, create short URLs, and manage their links.

# GEMINI.md - surl-fe

## Project Overview

This is a frontend application for a URL shortener service, built with **React** and **TypeScript**. It uses **Vite** for fast development and bundling. The application allows users to sign up, sign in, create short URLs, and manage their links.

**Key Technologies:**

*   **Framework:** React 19
*   **Language:** TypeScript
*   **Build Tool:** Vite
*   **Routing:** React Router DOM
*   **HTTP Client:** Axios
*   **Linting:** ESLint with TypeScript support

## Architecture

The project follows a standard React application structure, organizing files by feature and type.

*   `src/main.tsx`: The application's entry point. It sets up the `BrowserRouter`, `AuthProvider`, and renders the main `App` component.
*   `src/App.tsx`: Defines the main application routes, including public routes (like `/signin`, `/signup`) and private routes (the main dashboard) that require authentication.
*   `src/pages`: Contains the top-level page components for each route (e.g., `SignInPage.tsx`, `DashboardPage.tsx`).
*   `src/components`: Contains reusable UI components used across different pages. It includes layouts (`AuthLayout`, `MainLayout`) and smaller widgets (`ConfirmModal`, `ShortUrlTable`).
*   `src/services`: Manages all communication with the backend API.
    *   `api.ts`: Configures the central `axios` instance. It sets the base URL to `http://localhost:8080/api/` and includes an interceptor to automatically add the JWT `Authorization` header to all requests.
    *   `shortUrlService.ts` & `userService.ts`: Contain functions that use the configured axios instance to interact with the short URL and user-related API endpoints.
*   `src/commons/AuthContext.tsx`: Implements a React Context to provide global authentication state (e.g., the user token) and methods (`login`, `logout`) to the entire application.
*   `src/hooks`: Contains custom React hooks to encapsulate and reuse stateful logic.
    *   `useForm.ts`: A generic hook to manage form state.
    *   `useToast.ts`: A hook for displaying toast notifications.
*   `src/types`: Defines TypeScript interfaces for data structures like `IShortUrl` and `IAccount`.
*   `src/utils`: Provides miscellaneous utility functions.

## Building and Running

The project uses `npm` as the package manager. The following scripts are defined in `package.json`:

*   **Install dependencies:**
    ```bash
    npm install
    ```
*   **Run the development server:**
    Starts the app in development mode with hot-reloading.
    ```bash
    npm run dev
    ```
*   **Build for production:**
    Transpiles the TypeScript code and bundles the application into the `dist/` directory.
    ```bash
    npm run build
    ```
*   **Lint the code:**
    Analyzes the code for potential errors and style issues using ESLint.
    ```bash
    npm run lint
    ```
*   **Preview the production build:**
    Serves the contents of the `dist/` directory locally.
    ```bash
    npm run preview
    ```

## Development Conventions

*   **Authentication:** The app uses token-based authentication. The JWT is stored in `localStorage` and sent with every API request via an Axios interceptor.
*   **State Management:** Global authentication state is managed via `AuthContext`. Local component state is managed with `useState` and custom hooks.
*   **Styling:** Global styles are defined in `src/index.css`. Components are expected to use standard CSS classes.
*   **API Backend:** The frontend expects a backend API running at `http://localhost:8080`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/buzluk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
