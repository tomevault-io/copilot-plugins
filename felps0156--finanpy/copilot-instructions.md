## finanpy

> Finanpy is a personal finance management application built with Django (backend) and TailwindCSS (frontend). It follows a standard Model-View-Template (MVT) architecture, designed to be simple, visual, and functional. The application features a dark theme and is fully localized in Brazilian Portuguese (pt-BR).

# Project: Finanpy

## Project Overview

Finanpy is a personal finance management application built with Django (backend) and TailwindCSS (frontend). It follows a standard Model-View-Template (MVT) architecture, designed to be simple, visual, and functional. The application features a dark theme and is fully localized in Brazilian Portuguese (pt-BR).

**Key Features:**
*   **Accounts:** Manage bank accounts and balances.
*   **Transactions:** Record income and expenses.
*   **Categories:** Organize financial records.
*   **Dashboard:** Visual overview of financial health.
*   **Authentication:** User registration and login via email.

## Building and Running

### Prerequisites
*   Python 3.11+
*   Node.js & npm

### Setup Instructions

1.  **Backend Setup (Python):**
    ```bash
    # Create and activate virtual environment
    python -m venv .venv
    # Windows: .\.venv\Scripts\Activate
    # Linux/Mac: source .venv/bin/activate

    # Install dependencies
    pip install -r requirements.txt
    ```

2.  **Frontend Setup (Node.js):**
    ```bash
    # Install dependencies
    npm install

    # Build TailwindCSS (Run this when changing classes in templates)
    npm run build:css

    # Watch for CSS changes (Optional, for development)
    # npm run dev:css
    ```

3.  **Database & Server:**
    ```bash
    # Apply migrations
    python manage.py migrate

    # Run development server
    python manage.py runserver
    ```
    Access the app at `http://127.0.0.1:8000/`.

## Development Conventions

*   **Architecture:** 
    *   **Modular Apps:** Logic is separated into domain-specific apps: `accounts`, `categories`, `core`, `profiles`, `transactions`, `users`.
    *   **Templates:** Located in the root `templates/` directory, organized by app.
    *   **Static Files:** Tailwind input is in `assets/css/input.css`. Compiled output goes to `static/css/styles.css`.
*   **Code Style:** 
    *   Follow PEP8.
    *   Use single quotes `'` for strings where possible.
    *   Class-Based Views (CBVs) are preferred for CRUD operations.
*   **Database:** SQLite is the default database. All models should include `created_at` and `updated_at` timestamps.
*   **Language:** All user-facing text must be in **Brazilian Portuguese**.
*   **Design System:**
    *   **Theme:** Dark mode (`bg-zinc-950`).
    *   **Styling:** Use utility classes from TailwindCSS.
    *   **Colors:** Primary (Indigo/Purple), Success (Emerald), Error (Red).

## Directory Structure

*   `accounts/`, `categories/`, `transactions/`, `users/`, `profiles/`: Django apps containing models, views, and forms.
*   `core/`: Project settings and main URL configuration.
*   `templates/`: HTML templates extending `base.html`.
*   `assets/`: Source files for frontend assets (Tailwind CSS input).
*   `static/`: Compiled static files served by Django.
*   `docs/`: Project documentation and architecture details.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Felps0156)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Felps0156)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
