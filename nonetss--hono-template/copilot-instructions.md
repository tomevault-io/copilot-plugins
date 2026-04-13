## hono-template

> This project is a backend API service built with **Hono**, **Bun**, and **Drizzle ORM**. It serves as a media aggregation and scraping platform, interfacing with sources like Bunkr, Coomer, and Erome. The project utilizes OpenAPI for documentation and type-safe routing.

# Project Context: Hono Template

## Overview

This project is a backend API service built with **Hono**, **Bun**, and **Drizzle ORM**. It serves as a media aggregation and scraping platform, interfacing with sources like Bunkr, Coomer, and Erome. The project utilizes OpenAPI for documentation and type-safe routing.

## Key Technologies

- **Runtime:** [Bun](https://bun.sh/)
- **Web Framework:** [Hono](https://hono.dev/) (with `@hono/zod-openapi`)
- **Database:** PostgreSQL
- **ORM:** [Drizzle ORM](https://orm.drizzle.team/)
- **Validation:** [Zod](https://zod.dev/)
- **Documentation:** [Scalar](https://scalar.com/)

## Architecture & Project Structure

The project follows a modular, resource-based architecture within `src/api/v1`.

```text
src/
├── api/
│   └── v1/
│       ├── artist/         # Artist management endpoints
│       ├── favorite/       # User favorites management
│       └── search/         # Search functionality (Bunkr, Coomer, Erome)
│           ├── [source]/   # Source-specific implementations
│           └── [action]/
│               ├── *.handler.ts  # Request handling logic
│               ├── *.index.ts    # Route assembly
│               ├── *.route.ts    # OpenAPI route definition
│               └── *.schema.ts   # Zod schemas for validation
├── core/
│   ├── config.ts           # App configuration
│   └── services/           # Business logic services
├── db/
│   ├── index.ts            # Drizzle DB instance
│   └── models/             # Database schema definitions (*.table.ts)
├── lib/                    # Utility functions
└── index.ts                # Application entry point
```

### Coding Patterns & Conventions

- **Route Definition:** Routes are defined using `createRoute` from `@hono/zod-openapi` in `*.route.ts` files to ensure type safety and documentation generation.
- **Handlers:** Business logic resides in `*.handler.ts` files, keeping the router clean.
- **Database Access:** Uses Drizzle ORM's query builder (`db.select`, `db.query`).
- **External Proxies:** The application integrates with a "MediaFlow" service to proxy media streams (HLS/m3u8), as seen in `get.handler.ts`.

## Development Workflow

### Prerequisites

- **Bun** must be installed (`curl -fsSL https://bun.sh/install | bash`).
- **PostgreSQL** database.

### Setup

1.  Install dependencies:
    ```bash
    bun install
    ```
2.  Configure environment variables:
    ```bash
    cp .env.example .env
    # Edit .env with your DATABASE_URL
    ```

### Scripts

- **Start Development Server:**
  ```bash
  bun run dev
  ```
  - Server runs at: `http://localhost:3000`
  - API Documentation: `http://localhost:3000/scalar` or `http://localhost:3000/doc`

### Database Management (Drizzle)

- **Generate Migrations:**
  ```bash
  bunx drizzle-kit generate
  ```
- **Run Migrations:**
  ```bash
  bunx drizzle-kit migrate
  ```
- **Open Database Studio:**
  ```bash
  bunx drizzle-kit studio
  ```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Nonetss) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
