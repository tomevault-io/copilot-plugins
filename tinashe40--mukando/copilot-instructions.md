## mukando

> This project is a modern, full-stack web application called "Mukando," a community savings platform. It's built with a React frontend and a Supabase backend, designed to provide financial tools for groups and individuals. The application features user authentication, group management, loan requests, payment processing, and data analytics.

# GEMINI.md

## Project Overview

This project is a modern, full-stack web application called "Mukando," a community savings platform. It's built with a React frontend and a Supabase backend, designed to provide financial tools for groups and individuals. The application features user authentication, group management, loan requests, payment processing, and data analytics.

**Key Technologies:**

*   **Frontend:** React 18, Vite, Redux Toolkit, Tailwind CSS, React Router v6
*   **Backend:** Supabase (PostgreSQL, Authentication, Storage)
*   **Styling:** Tailwind CSS with a custom design system
*   **Data Visualization:** D3.js and Recharts
*   **Form Management:** React Hook Form

## Building and Running

### Prerequisites

*   Node.js (v14.x or higher)
*   npm or yarn
*   A Supabase project with the necessary environment variables set in a `.env` file.

### Key Commands

*   **Install Dependencies:**
    ```bash
    npm install
    # or
    yarn install
    ```

*   **Start the Development Server:**
    ```bash
    npm start
    # or
    yarn start
    ```
    The application will be available at `http://localhost:4028`.

*   **Build for Production:**
    ```bash
    npm run build
    ```
    This will create a `build` directory with the production-ready assets.

*   **Preview the Production Build:**
    ```bash
    npm run serve
    ```

## Development Conventions

*   **Routing:** New pages and routes should be added to `src/Routes.jsx`.
*   **Styling:** The project uses Tailwind CSS with a custom theme defined in `tailwind.config.js`. Utility classes should be used for styling components.
*   **State Management:** Redux Toolkit is used for global state management. For component-level state, use React hooks.
*   **Authentication:** Authentication is handled by Supabase. The `AuthContext` at `src/contexts/AuthContext.jsx` provides user data and authentication methods throughout the application.
*   **Database:** The database schema is managed through Supabase migrations located in the `supabase/migrations` directory.
*   **API Interaction:** The Supabase client is initialized in `src/lib/supabase.js`. This file also contains helper functions for interacting with the database.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Tinashe40) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
