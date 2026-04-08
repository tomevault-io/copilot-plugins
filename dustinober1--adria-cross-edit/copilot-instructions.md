## adria-cross-edit

> Adria Cross Edit is a full-stack web application for a personal styling business. It provides a platform for:

# Adria Cross Edit - Project Context

## Project Overview
Adria Cross Edit is a full-stack web application for a personal styling business. It provides a platform for:
- **Client Services:** Booking appointments, intake forms (`intake-form.html`).
- **Clothing Matcher:** A tool for clients to upload wardrobe items and get style matches (`clothing-matcher/`).
- **Content:** A static-generated blog system (`blog/`).
- **Administration:** Admin panel for managing appointments, invoices, and content (`admin.html`).
- **E-commerce:** Integration with Square for payments and invoicing.

## Technology Stack
- **Runtime:** Node.js
- **Framework:** Express.js
- **Database:** 
  - **Primary (Production):** PostgreSQL
  - **Development/Fallback:** SQLite (`adria_cross.db`)
  - **ORM/Query:** `better-sqlite3` (SQLite) / `pg` (Postgres) w/ custom unified query wrapper.
- **Frontend:** Vanilla HTML5, CSS3, JavaScript. 
- **Authentication:** `express-session`, `bcryptjs` (Moving to Passport.js for OAuth).
- **APIs:** Square Node.js SDK, Nodemailer.
- **Infrastructure:** Docker, Nginx, Render.yaml.

## Directory Structure
- **Root:** Main HTML pages (`index.html`, `about.html`, etc.), `server.js` (Entry point).
- **`/blog`:** Generated static blog posts and index.
- **`/clothing-matcher`:** Source for the wardrobe matching tool (CSS/JS/HTML).
- **`/migrations`:** SQL files for database schema updates.
- **`/plans`:** Documentation and implementation plans (e.g., `email-integration.md`, `implementation_plan.md`).
- **`/scripts`:** Utility scripts (`migrate.js`, `link-checker.js`).
- **`/tests`:** Jest test files.
- **`/uploads`:** User-uploaded content (images).

## Development Workflow

### Setup & Run
1.  **Install Dependencies:**
    ```bash
    npm install
    ```
2.  **Environment Variables:**
    -   Required: `DATABASE_URL` (or sqlite path), `SESSION_SECRET`, `ADMIN_PASSWORD`.
    -   Optional: `SQUARE_ACCESS_TOKEN`, `EMAIL_HOST`, etc.
3.  **Database Initialization:**
    -   Managed via `scripts/migrate.js`.
    -   Runs automatically on server start if configured.
4.  **Start Server:**
    ```bash
    npm start         # Runs node server.js
    # OR
    node server.js
    ```
5.  **Run Tests:**
    ```bash
    npm test          # Runs Jest
    ```

### Code Conventions
-   **Backend:** CommonJS modules (`require`). Async/Await patterns.
-   **Database:** unified `pool.query(sql, params)` interface to support both SQLite and Postgres.
-   **Frontend:** Standard HTML/CSS/JS. No heavy frameworks (React/Vue) currently visible in root, though specific tools may exist in subdirectories.
-   **Logging:** Winston logger (`logger.js`).

## Key Features & Logic
-   **Hybrid Database Support:** `server.js` detects `DATABASE_URL` protocol to switch between `sqlite` and `pg`.
-   **Availability System:** Custom logic for "Availability Overrides" vs "Weekly Config".
-   **Clothing Matcher:** Algorithm uses color theory (complementary/analogous) and style tags to score matches.

## Current Status
-   **Active Task:** Implementation of OAuth (Google/Apple) as per `implementation_plan.md`.
-   **Recent Changes:** Updates to Privacy Policy and Terms of Service (Legal compliance).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dustinober1)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dustinober1)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
