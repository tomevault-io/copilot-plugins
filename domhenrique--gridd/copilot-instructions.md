## gridd

> This is a **Gridd360 Asset Manager**, a web application designed for centralizing, organizing, and managing digital assets. It's built with **React** and **TypeScript**, using **Vite** as the build tool. The backend is powered by **Supabase**, which handles data storage, authentication, and more. The UI is styled with **Bootstrap** and **Lucide Icons**.

# GEMINI.md

## Project Overview

This is a **Gridd360 Asset Manager**, a web application designed for centralizing, organizing, and managing digital assets. It's built with **React** and **TypeScript**, using **Vite** as the build tool. The backend is powered by **Supabase**, which handles data storage, authentication, and more. The UI is styled with **Bootstrap** and **Lucide Icons**.

The application features a hybrid authentication system (Google OAuth 2.0 and Email/Senha), hierarchical folder management with granular access control (RBAC), and detailed activity auditing.

## Building and Running

### Prerequisites

*   Node.js (v18+)
*   A Supabase account
*   A Google Cloud Project (for OAuth)

### Installation and Execution

1.  **Install dependencies:**
    ```bash
    npm install
    ```

2.  **Configure the environment:**
    *   Copy the `.env.example` file to a new file named `.env.local`:
        ```bash
        cp .env.example .env.local
        ```
    *   Fill in the required keys in `.env.local`:
        ```env
        # Supabase
        VITE_SUPABASE_URL=https://your-url.supabase.co
        VITE_SUPABASE_ANON_KEY=your-anon-key

        # Google OAuth
        VITE_GOOGLE_CLIENT_ID=your-client-id.apps.googleusercontent.com

        # Application
        VITE_APP_URL=http://localhost:5173
        ```

3.  **Run the development server:**
    ```bash
    npm run dev
    ```

4.  **Build for production:**
    ```bash
    npm run build
    ```

5.  **Preview the production build:**
    ```bash
    npm run preview
    ```

## Development Conventions

*   **Components:** Reusable components are located in the `/components` directory.
*   **Pages:** Main application pages are in the `/pages` directory.
*   **Services:** Business logic and integrations (Supabase, Google Photos) are in the `/services` directory.
*   **Database:** SQL scripts for migrations and database structure are in the `/database` directory.
*   **Types:** Global TypeScript type definitions are in the `/types` directory.
*   **Authentication:** The application uses Google's Implicit Flow for OAuth 2.0 to avoid CORS issues. Tokens are persisted in Supabase.
*   **Permissions:** Access to folders is controlled via a Role-Based Access Control (RBAC) system (Viewer, Editor, Admin) with inheritance for subfolders.
*   **Logging:** New activities should be logged using the `DataService.logActivity(userId, action, targetName)` method.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DomHenrique)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DomHenrique)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
