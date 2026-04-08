## time-tracker

> This is a time-tracking web application built with Vue.js 3 and Vite. It uses

# GEMINI.md

## Project Overview

This is a time-tracking web application built with Vue.js 3 and Vite. It uses
r components, Pinia for state management, and TanStack Vue Query for data fetching.
The backend is powered by PocketBase, a self-hosted backend service. The application
is set up for internationalization using Paraglide JS.

The application allows users to track time spent on different projects. It
includes features for adding projects, viewing tracked time entries, and filtering
by day.

## Building and Running

### Prerequisites

- Node.js and pnpm

### Installation

```bash
pnpm install
```

### Development

To run the application in development mode, use the following command. This will
start the Vite development server and the PocketBase backend concurrently.

```bash
pnpm dev
```

The application will be available at `http://localhost:5173`.

### Building

To build the application for production, use the following command:

```bash
pnpm build
```

This will create a `dist` directory with the production-ready assets.

### Seeding the Database

The project includes a script to seed the database with mock data. To run the seed
script, use the following command:

```bash
pnpm seed:db
```

## Development Conventions

### Linting

The project uses ESLint for code linting. To check for linting errors, run:

```bash
pnpm lint
```

To automatically fix linting errors, run:

```bash
pnpm lint:fix
```

### Type Checking

The project uses TypeScript for static type checking. To check for type errors, run:

```bash
pnpm typecheck
```

### Routing

The project uses `unplugin-vue-router` for file-based routing. Pages are located
in the `src/pages` directory. The routes are generated automatically based on the
file structure.

### State Management

The project uses Pinia for state management. Stores are located in the
`src/lib/stores` directory.

### Backend

The backend is a PocketBase instance located in the `pocketbase` directory. The
data schema is defined in the PocketBase UI, which can be accessed at
`http://localhost:8090/_/` when the development server is running.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NillsvanLimbeek)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/NillsvanLimbeek)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
