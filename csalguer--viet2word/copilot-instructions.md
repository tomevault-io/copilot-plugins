## viet2word

> A modern web application for searching Vietnamese vocabulary and reading text with integrated dictionary lookups.

# GEMINI Summary

## Project: Vietnamese-English Dictionary & Reader
A modern web application for searching Vietnamese vocabulary and reading text with integrated dictionary lookups.

## System Overview
The project uses a **Client-Server** architecture:

1.  **Backend (Python):** A FastAPI service (`backend/`) that serves dictionary data from a SQLite/PostgreSQL database.
    -   **API Root:** `http://127.0.0.1:8000/api/v1`
    -   **Docs:** `http://127.0.0.1:8000/docs`
2.  **Frontend (React):** A React 19 application (`frontend/`) that consumes the backend API via a Vite proxy.
    -   **Dev Server:** `http://localhost:5173` (Proxies `/api/v1` -> Backend)

## Key Tech Stack
- **Backend:** Python 3.12+, FastAPI, SQLite/PostgreSQL, SQLAlchemy/Alembic.
- **Frontend:** React 19, Vite 7, TanStack Query, TanStack Router.
- **State:** Zustand (`src/lib/store.ts`) for client state (Saved Vocab).
- **Styling:** Tailwind CSS + Chakra UI (Custom Theme).
- **Testing:** Storybook (Components & Interactions), Vitest (Unit), Playwright (E2E).

## Developer Guide
### Getting Started

1.  **Backend Setup:**
    ```bash
    cd backend
    ./manage.sh init   # Initialize DB
    ./manage.sh start  # Start API Server
    ```

2.  **Frontend Setup:**
    ```bash
    cd frontend
    pnpm install
    pnpm run dev       # Start Frontend Dev Server
    ```

3.  **Testing:**
    ```bash
    cd frontend
    pnpm run storybook # Start Component/Interaction Tests
    ```

### Key Directories
- `backend/`: FastAPI application and database logic.
- `frontend/src/features/`: Domain-specific frontend logic.
- `frontend/src/lib/`: Core utilities and API clients (`api.ts`).
- `data/`: Raw dictionary datasets (JSONL).

### Common Tasks
- **Adding Data:** Use `backend/manage.sh ingest <file.jsonl>` to add new dictionary entries.
- **UI Changes:** Components are developed and tested in **Storybook** first (`src/components/**/*.stories.tsx`).
- **Testing:** Always verify changes in Storybook before committing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/csalguer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
