## textutils

> **TextUlis** is a React application built with Vite designed to perform various utility operations on text. It allows users to analyze text (word/character count, reading time) and manipulate it (uppercase/lowercase conversion, clearing, copying, removing extra spaces). The application features a dynamic theming system with Dark and Green modes.

# Project Overview

**TextUlis** is a React application built with Vite designed to perform various utility operations on text. It allows users to analyze text (word/character count, reading time) and manipulate it (uppercase/lowercase conversion, clearing, copying, removing extra spaces). The application features a dynamic theming system with Dark and Green modes.

## Tech Stack

*   **Frontend Framework:** React 18
*   **Build Tool:** Vite
*   **Styling:** Bootstrap 5.3 (via CDN), CSS Modules, and inline styles for theming.
*   **Routing:** `react-router-dom`
*   **Icons:** `bootstrap-icons`
*   **Linting:** ESLint

## Building and Running

The project uses standard `npm` scripts defined in `package.json`.

*   **Install Dependencies:**
    ```bash
    npm install
    ```
*   **Start Development Server:**
    ```bash
    npm run dev/
    ```
*   **Build for Production:**
    ```bash
    npm run build/m
    ```
*   **Preview Production Build:**
    ```bash
    npm run preview
    ```
*   **Lint Code:**
    ```bash
    npm run lint
    ```

## Key Files and Components

*   **`src/App.jsx`**: The main application component. It handles routing, global state for themes (`light`, `dark`, `green`), and the alert system.
*   **`src/components/Textform.jsx`**: The core component containing the text area and action buttons (Uppercase, Lowercase, Copy, etc.). It also displays text summaries.
*   **`src/components/Navbar.jsx`**: The top navigation bar containing links and the theme toggle switches.
*   **`src/components/Alert.jsx`**: A component to display temporary success/info messages to the user.
*   **`index.html`**: The entry HTML file, notably including Bootstrap CSS and JS via CDN.

## Development Conventions

*   **Component Structure:** Components are located in `src/components/`.
*   **Styling:** 
    *   Heavy reliance on Bootstrap utility classes (e.g., `container`, `btn`, `form-control`).
    *   Dynamic theming is implemented via inline styles manipulating `backgroundColor` and `color` based on `mode` props.
*   **State Management:** `useState` is used for managing text input, alerts, and application themes.
*   **Naming:** 
    *   Components use PascalCase (e.g., `Textform`).
    *   Props and functions show a mix of conventions (e.g., `bg`, `modeChanger`, `Mode`, `Changemode`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Shahzishaheer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
