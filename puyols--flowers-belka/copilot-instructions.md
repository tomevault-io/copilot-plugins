## flowers-belka

> This document provides a comprehensive overview of the "Flowers Belka" project, designed to be used as a primary context source for the Gemini AI assistant.

# GEMINI.md - Your AI Assistant's Guide to this Project

This document provides a comprehensive overview of the "Flowers Belka" project, designed to be used as a primary context source for the Gemini AI assistant.

## Project Overview

This is a full-stack e-commerce project for a flower shop named "Flowers Belka". It consists of two main parts:

*   **Backend:** An OpenCart 3.x installation serves as the backend. It manages products, orders, and other e-commerce functionalities. The project is configured to connect to a MySQL database.
*   **Frontend:** A modern Next.js 14 application provides the user-facing storefront. It's written in TypeScript and styled with Tailwind CSS.

The two parts are connected via a custom REST API implemented in PHP (`api_products.php`, `api_analytics.php`, etc.).

### Key Technologies

*   **Backend:** OpenCart 3.x, PHP 7.4+, MySQL
*   **Frontend:** Next.js 14, React, TypeScript, Tailwind CSS
*   **Integration:** Custom REST API (PHP)

## Building and Running

### Backend (OpenCart)

1.  **Prerequisites:** A PHP and MySQL environment, like XAMPP, is required.
2.  **Database Setup:**
    *   Create a MySQL database named `flowers_belka_new`.
    *   Import the `localhost.sql` file into the database.
3.  **Configuration:**
    *   The main configuration file is `config.php`. It's pre-configured for a local environment with the following database credentials:
        *   **User:** `flowers_user`
        *   **Password:** `secure_password_123`
        *   **Database:** `flowers_belka_new`
4.  **Running:**
    *   The backend is intended to be run with a web server like Apache. The `XAMPP_SETUP.md` file provides detailed instructions for setting up the project with XAMPP.
    *   Alternatively, you can use the built-in PHP server:
        ```bash
        php -S localhost:8080
        ```

### Frontend (Next.js)

1.  **Prerequisites:** Node.js 18+
2.  **Installation:**
    ```bash
    cd flowers-belka-nextjs
    npm install
    ```
3.  **Running in Development Mode:**
    ```bash
    npm run dev
    ```
    The frontend will be available at `http://localhost:3001`.
4.  **Building for Production:**
    ```bash
    npm run build
    ```
5.  **Running in Production Mode:**
    ```bash
    npm run start
    ```

## Development Conventions

*   The frontend code is located in the `flowers-belka-nextjs` directory.
*   The backend code is in the root directory, with the main application logic in the `catalog` and `admin` directories.
*   The custom API files are in the root directory (e.g., `api_products.php`).
*   The `README.md` file provides a good overview of the project structure and API endpoints.
*   The `XAMPP_SETUP.md` file is the primary source of truth for setting up the local development environment.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/puyols)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/puyols)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
