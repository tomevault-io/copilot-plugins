## capao-do-tesouro

> This project is a web-based treasure hunting game called "Capão do Tesouro". It is built with a modern frontend stack that includes:

# Project: Capão do Tesouro

## Project Overview

This project is a web-based treasure hunting game called "Capão do Tesouro". It is built with a modern frontend stack that includes:

*   **Framework:** React with TypeScript
*   **Build Tool:** Vite
*   **Styling:** Tailwind CSS
*   **Mapping:** Leaflet and React-Leaflet for interactive maps
*   **Authentication and Database:** Firebase
*   **State Management:** Zustand
*   **Routing:** React Router
*   **PWA:** The application is a Progressive Web App.

The application seems to be structured around a core gameplay loop involving quests, a map, and a leaderboard. The folder structure is well-organized, with clear separation of concerns between UI components, features, services, and state management.

## Building and Running

To get the project up and running, you can use the following npm scripts defined in `package.json`:

*   **`npm run dev`**: Starts the development server with Hot Module Replacement (HMR). This is the primary command for development.
*   **`npm run build`**: Builds the application for production. This will create a `dist` folder with the optimized and minified assets.
*   **`npm run lint`**: Lints the codebase using ESLint to enforce code quality and style.
*   **`npm run preview`**: Serves the production build locally to preview the final application.

**TODO:** Add instructions for setting up Firebase credentials. The file `capao-do-tesouro-firebase-adminsdk-fbsvc-a9a96c021e.json` suggests that Firebase is already configured, but it would be beneficial to document the setup process for new developers.

## Development Conventions

Based on the file structure and the code examples, the following development conventions can be inferred:

*   **Component-Based Architecture:** The application is built using a component-based architecture, with a clear distinction between UI components and feature components.
*   **State Management:** Zustand is used for global state management. It is recommended to create separate stores for different parts of the application (e.g., `authStore`, `questStore`).
*   **Styling:** Tailwind CSS is the primary styling solution. Utility-first classes are used to style the components.
*   **File Naming:** Files are named using PascalCase for components (e.g., `CheckInButton.tsx`) and camelCase for other files (e.g., `authStore.ts`).
*   **Folder Structure:** The `src` folder is organized by feature and type, which makes it easy to navigate and locate files.
*   **Authentication:** Authentication is handled by Firebase. The `useAuth` hook provides a simple way to access the user's authentication state.
*   **Routing:** React Router is used for routing. It is recommended to define the routes in a central location and use the `ProtectedRoute` component to protect routes that require authentication.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gabrielmaialva33)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gabrielmaialva33)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
