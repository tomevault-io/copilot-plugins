## dev-environment

> Docker dev environment – frontend/backend usually running with hot reload


# Dev Environment (Docker)

- **Frontend and backend run in Docker** with hot reload (frontend and backend dirs are volume-mounted).
- **Assume they are already running.** You can check the app in the browser or hit APIs without starting anything.
- **Only if services are not running:** run `docker compose up -d`. Do not start frontend/backend manually outside Docker for normal dev.
- **Frontend URL:** Access the frontend/backend at `https://localhost:4433/` (via nginx reverse proxy, self-signed certificate)

## Path Conventions

- **Always use generic paths in examples and documentation** - use `~` (tilde) for home directory instead of user-specific absolute paths
- This applies to `.env.example`, documentation, comments, and any example code.

---
> Source: [mycelia-tech/mycelia](https://github.com/mycelia-tech/mycelia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
