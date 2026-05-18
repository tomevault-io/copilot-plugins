## 08-testing-and-deployment

> - Implement **unit tests** (e.g., using `pytest` for backend, Jest/React Testing Library for frontend).


### Testing and Deployment

- Implement **unit tests** (e.g., using `pytest` for backend, Jest/React Testing Library for frontend).
- Ensure proper input validation (Pydantic on backend, Zod/react-hook-form on frontend) and error handling.
- **Deployment**: Deployed as a Databricks App via `databricks bundle deploy` using `api/app.yaml`.
- **Development Server**: When developing locally (localhost is used), the FastAPI/Uvicorn and Vite server run in dev mode with auto-reload enabled. File changes automatically restart/reload the server - no manual restart needed. **IMPORTANT**: NEVER RESTART SERVER PROCESSES!
- Backend server logs are in `/tmp/backend.log`, frontend server logs in `/tmp/frontend.log`
- **Development Ports**: Backend API runs on port 8000, Frontend UI runs on port 3000.
- Always use `hatch -e dev run ...` to run Python snippets
- IMPORTANT: The backend server (Python) logs are in `/tmp/backend.log` and the frontend (Vite) in `/tmp/frontend.log. Read them as needed.

---
> Source: [databrickslabs/ontos](https://github.com/databrickslabs/ontos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
