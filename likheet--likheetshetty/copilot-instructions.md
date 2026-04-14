## likheetshetty

> This project is a personal blog website for Likheet Shetty. It's built with Node.js and Express, and it uses SQLite for the database. The blog has a simple interface for reading posts, an admin panel for managing content, and a subscription system with email notifications.

# GEMINI.md

## Project Overview

This project is a personal blog website for Likheet Shetty. It's built with Node.js and Express, and it uses SQLite for the database. The blog has a simple interface for reading posts, an admin panel for managing content, and a subscription system with email notifications.

**Key Technologies:**

*   **Backend:** Node.js, Express.js
*   **Database:** SQLite
*   **Frontend:** HTML, CSS, JavaScript
*   **Email:** Nodemailer

**Architecture:**

The application is a monolithic Node.js application. The `server.js` file contains the majority of the application logic, including the Express server, API routes, database initialization, and email functionality. The frontend is composed of static HTML, CSS, and JavaScript files served from the `public` directory.

## Building and Running

*   **Install dependencies:**
    ```bash
    npm install
    ```

*   **Run in development mode (with auto-reloading):**
    ```bash
    npm run dev
    ```

*   **Run in production mode:**
    ```bash
    npm start
    ```

*   **Access the application:**
    *   **Blog:** [http://localhost:3000](http://localhost:3000)
    *   **Admin Panel:** [http://localhost:3000/admin](http://localhost:3000/admin)

## Development Conventions

*   **Code Style:** The code follows standard JavaScript conventions. There is no linter configured, but the code is clean and well-formatted.
*   **Testing:** There are no tests in the project. The `package.json` has a `test` script that currently does nothing.
*   **Database:** The database schema is defined and created in `server.js`. The database file is `blog.db`.
*   **Environment Variables:** The application uses a `.env` file for configuration. An example is provided in `.env.example`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Likheet)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Likheet)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
