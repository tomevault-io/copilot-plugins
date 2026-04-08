## sci-fi-viewer

> This project is a React-based web application called "Sci-Fi Tracker". It allows users to discover, rate, and track science fiction movies and TV series. The application can fetch data from The Movie Database (TMDB) API or use a curated set of mock data as a fallback. It features a dark, modern user interface and persists user data (ratings and comments) in the browser's local storage.

# GEMINI.md

## Project Overview

This project is a React-based web application called "Sci-Fi Tracker". It allows users to discover, rate, and track science fiction movies and TV series. The application can fetch data from The Movie Database (TMDB) API or use a curated set of mock data as a fallback. It features a dark, modern user interface and persists user data (ratings and comments) in the browser's local storage.

The main technologies used are:
*   **React:** For building the user interface.
*   **react-scripts:** For managing the project's build process.
*   **lucide-react:** For icons.

The application's main logic is contained within the `src/App.js` file. This file defines the main `SciFiTracker` component, which manages the application's state, fetches data, and renders the UI. The UI is composed of a `MovieCard` component that displays individual movie/TV series information and allows for user interaction.

## Building and Running

To build and run this project, you will need to have Node.js and npm installed.

1.  **Install dependencies:**
    ```bash
    npm install
    ```

2.  **Run the development server:**
    ```bash
    npm start
    ```
    This will start the application in development mode and open it in your browser at `http://localhost:3000`.

3.  **Build for production:**
    ```bash
    npm run build
    ```
    This will create a production-ready build of the application in the `build` directory.

## Development Conventions

*   **State Management:** The application uses React hooks (`useState`, `useEffect`) for state management.
*   **Data Persistence:** User ratings and comments are stored in the browser's local storage.
*   **Styling:** The project uses inline styles within the `src/App.js` file.
*   **API Interaction:** The application fetches data from the TMDB API using the `fetch` API. It includes a CORS proxy to work around browser restrictions.
*   **Mock Data:** A set of mock data is included in `src/App.js` for offline development and as a fallback if the TMDB API is unavailable.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ben-newman10)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ben-newman10)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
