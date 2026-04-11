## superapp-rust

> This repository contains the source code for "SuperApp," a comprehensive platform inspired by WeChat. It is built with a focus on security, performance, and scalability, utilizing a microservices-style architecture in a Rust monorepo. The backend is powered by Rust, while the frontend is a Next.js application.

# SuperApp Rust Project

## Project Overview

This repository contains the source code for "SuperApp," a comprehensive platform inspired by WeChat. It is built with a focus on security, performance, and scalability, utilizing a microservices-style architecture in a Rust monorepo. The backend is powered by Rust, while the frontend is a Next.js application.

**Backend (Rust):**

*   **Architecture:** The backend follows Clean Architecture and Domain-Driven Design (DDD) principles. The codebase is organized into a workspace with multiple crates, each responsible for a specific domain (e.g., `app-api`, `app-core`, `app-social`, `app-auth`).
*   **Key Technologies:**
    *   **Web Framework:** Axum
    *   **Async Runtime:** Tokio
    *   **Database:** PostgreSQL (with SQLx)
    *   **Caching:** Redis
    *   **Blockchain:** Solana (with Anchor) for DAO features.
    *   **Authentication:** JWT-based authentication.
    *   **AI/ML:** ONNX for machine learning model inference.

**Frontend (Next.js):**

*   **Framework:** Next.js 14
*   **Language:** TypeScript
*   **Styling:** Tailwind CSS
*   **State Management:** Zustand
*   **Data Fetching:** React Query and Axios

## Building and Running

### Backend (Rust)

There are three primary ways to run the backend:

1.  **Full Docker Setup (Easiest):** This method runs the entire application, including the API, in Docker containers.
    ```bash
    docker-compose up -d
    ```

2.  **Docker for Dependencies + Local API (Recommended for Development):** This is the recommended approach for backend development. It runs PostgreSQL and Redis in Docker, while the application code runs directly on your local machine.
    ```bash
    # Start PostgreSQL and Redis
    docker-compose -f docker-compose.dev.yml up -d

    # Run the API server
    cargo run -p app-api
    ```

3.  **Fully Local Setup:** This method requires you to install and manage all dependencies (Rust, PostgreSQL, Redis) on your local machine.
    ```bash
    # Run the setup script (make sure it's executable)
    ./scripts/setup.sh

    # Run the API server
    cargo run -p app-api
    ```

The backend server will be available at `http://localhost:8080`.

### Frontend (Next.js)

To run the frontend development server:

```bash
cd frontend
npm install
npm run dev
```

The frontend will be available at `http://localhost:3000`.

## Development Conventions

### Backend (Rust)

*   **Formatting:** Code is formatted with `rustfmt`. To format the entire workspace, run:
    ```bash
    cargo fmt --all
    ```
*   **Linting:** `clippy` is used for linting. To check for lints, run:
    ```bash
    cargo clippy --all-targets --all-features -- -D warnings
    ```
*   **Testing:** The project uses a combination of unit and integration tests. To run all tests in the workspace:
    ```bash
    cargo test --workspace
    ```
*   **Continuous Integration:** The CI pipeline, defined in `.github/workflows/ci.yml`, enforces formatting, linting, and testing on every push and pull request.

### Frontend (Next.js)

*   **Linting:** The frontend uses ESLint for code quality.
    ```bash
    npm run lint
    ```
*   **Type Checking:** TypeScript is used for static type checking.
    ```bash
    npm run type-check
    ```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Adylitto)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Adylitto)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
