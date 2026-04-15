## love-diary-v2

> This is **Love Diary**, a social web application created for the author and their long-distance partner. It's a React-based single-page application (SPA) designed to be a private space for a couple to share memories, and stay connected.

# Project Overview

This is **Love Diary**, a social web application created for the author and their long-distance partner. It's a React-based single-page application (SPA) designed to be a private space for a couple to share memories, and stay connected.

## Main Technologies

*   **Frontend:** React, Material-UI, styled-components
*   **Data Fetching & Caching:** React Query
*   **State Management:** React Query and React Context (`settingsContext`).
*   **Backend:** Airtable is used as a lightweight database.
*   **Internationalization:** i18next
*   **Routing:** React Router

## Architecture

The application is structured as a typical Create React App project.
-   `src/pages`: Contains the main page components for each feature.
-   `src/components`: Contains reusable UI components.
-   `src/services/airtable`: Contains the logic for interacting with the Airtable API.
-   `src/hooks`: Contains custom hooks for shared logic.
-   `src/app.js`: The main application component, which sets up the theme, routing, and global context.
-   `src/routes.js`: Defines the application's routes.

# Building and Running

1.  **Install dependencies:**
    ```bash
    npm install
    ```
2.  **Run the development server:**
    ```bash
    npm start
    ```
    The application will be available at http://localhost:3000.

3.  **Build for production:**
    ```bash
    npm build
    ```

# Development Conventions

*   **Styling:** The project uses a combination of Material-UI components and `styled-components` for custom styling.
*   **Data Fetching:** Data fetching from Airtable is primarily handled by `react-query`. Asynchronous operations are defined in the `src/services/airtable` directory and called from components using the `useQuery` and `useMutation` hooks.
*   **State Management:** The project uses a combination of React Query and React Context for state management. React Query is the primary tool for managing server state (all data fetched from Airtable). React Context (`settingsContext`) is used for global client state that is not persisted on the server, such as UI state and session information.
*   **Routing:** The application uses `react-router-dom` for client-side routing. Authenticated routes are protected and only accessible after logging in.
*   **Internationalization:** The app supports English and Chinese, with translations managed in `public/assets/i18n/translations`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/human2l)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/human2l)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
