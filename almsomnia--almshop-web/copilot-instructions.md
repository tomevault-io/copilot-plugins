## almshop-web

> This is the frontend repository for the **Almshop** project. It is built using **Nuxt 4** and **Vue 3**, leveraging a **pnpm workspace** structure to organize the codebase into a modular "Layers" architecture.

# Project Context: Almshop Frontend

## Overview

This is the frontend repository for the **Almshop** project. It is built using **Nuxt 4** and **Vue 3**, leveraging a **pnpm workspace** structure to organize the codebase into a modular "Layers" architecture.

## Architecture

The project is structured as a monorepo using pnpm workspaces with a tiered "Layer" inheritance model:

- **`app/`**: The main Nuxt application entry point. It consumes and orchestrates functionality from the domain layers.
- **`layers/base`**: The foundation layer. Contains low-level utilities, base composables, and fundamental shared logic used by all other layers.
- **`layers/core`**: Infrastructure layers. Provides shared cross-cutting concerns like UI design systems, API fetching clients, and global server utilities.
- **`layers/domains`**: Feature-specific layers. Encapsulates business logic, stores, components, and server-side logic for specific application domains (e.g., Auth, Products, Users).
- **`types/`**: Centralized TypeScript definitions for API responses and Data Transfer Objects (DTOs), shared across the entire workspace.
- **`utils/`**: Internal development utilities and tooling, such as the layer generator CLI.

## Key Technologies

- **Framework:** Nuxt 4, Vue 3
- **Package Manager:** pnpm (with workspaces and catalogs)
- **Styling:** Tailwind CSS (via Nuxt UI)
- **Language:** TypeScript

## Development Workflow

### Prerequisites

- Node.js (Ensure a compatible version for Nuxt 4)
- pnpm

### Setup

Install project dependencies from the root directory:

```bash
pnpm install
```

### Running the Application

To start the development server (runs the `app` package):

```bash
pnpm dev
```

Access the application at `http://localhost:3000`.

### Building for Production

To build the application:

```bash
pnpm build
```

### Layer Generation

To scaffold a new layer (core or domain):

```bash
pnpm gen:layer
```

### Code Quality

Format code using Prettier:

```bash
pnpm format
```

### Maintenance

To clean all `node_modules` and `.nuxt` directories across workspaces:

```bash
pnpm clean:all
```

## Directory Structure

- `package.json`: Root configuration defining workspaces and global scripts.
- `pnpm-workspace.yaml`: Defines workspace packages and catalogs.
- `app/`: Main application source.
- `layers/`: Shared feature and UI layers.
- `types/`: Centralized global TypeScript type definitions.
- `utils/`: Development utility packages.

## General Instruction

- Before you trying to update or modify a task, always make sure to read it first and understand any logic and/or patterns used in the code.
- When you generate new code, always follow the existing code style and patterns used in the project.
- When you generate new code, always make a proper documentation for the code.
- When you generate new code, always make a proper error handling.
- **Confirm Dangerous Commands**: If the intended command is potentially destructive (e.g., rm, mv, sudo, systemctl), you **must** explicitly preface the command proposal with a warning: 'WARNING: POTENTIALLY DESTRUCTIVE ACTION REQUIRED'.
- **Do not** create a git commit and/or push to remote repository unless explicitly asked to do so.
- **Do not** build the application unless explicitly asked to do so.

## Code Conventions

- Use Nuxt UI as primary UI component library.
- Always use TypeScript unless explicitly to use another language.
- Always use PascalCase for component names.
- Always use kebab-case for component's prop and emit names.
- Always use camelCase for variables, methods, utils and composables names.
- Always use kebab-case for file names and directories.
- Prefer nullish coalescing operator (`??`) over logical OR (`||`) for default values.
- **Utils naming**: Utility functions are prefixed with `$` (e.g., `$formatCurrency`) and are auto-imported.
- **Composables naming**: Composables are prefixed with `use` (e.g., `useAuth`) and are auto-imported.
- **Layer Extension Rules**:
   - `./app` is only allowed to extend Nuxt apps located in `./layers/domains`.
   - `./layers/domains` are allowed to extend Nuxt apps located in `./layers/core`.
   - `./layers/core` are allowed to extend Nuxt apps located in `./layers/base`.
   - `./layers/base` is not allowed to extend any layers.
- **Formatting**: Always format code using Prettier (`pnpm format`) after completing changes.
- **Centralized Types**:
   - Global types are defined in `types/` directory (e.g., `API`, `DTO` namespaces).
   - These types are integrated into each Nuxt application via `nuxt.config.ts` by adding the appropriate relative path to the `include` array of the TypeScript configuration.
   - They are globally available and do not require explicit imports.

## llms.txt

Use these LLMs for different tasks:

- **Nuxt**: https://nuxt.com/llms.txt
- **Vue**: https://vuejs.org/llms.txt
- **Nuxt UI**: https://ui.nuxt.com/llms.txt
- **Tailwind CSS**: https://raw.githubusercontent.com/rgfx/tailwind-llms/refs/heads/main/tailwind-llms.txt

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/almsomnia)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/almsomnia)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
