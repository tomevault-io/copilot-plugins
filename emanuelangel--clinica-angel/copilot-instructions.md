## clinica-angel

> Use these skills for detailed patterns on-demand:

# Repository Guidelines

## Available Skills

Use these skills for detailed patterns on-demand:

| Skill                   | Description                                            | URL                                                      |
| ----------------------- | ------------------------------------------------------ | -------------------------------------------------------- |
| `skill-creator`         | Create new AI agent skills                             | [SKILL.md](.agent/skills/skill-creator/SKILL.md)         |
| `nunjucks-views`        | Patterns for Nunjucks templates and UI                 | [SKILL.md](.agent/skills/nunjucks-views/SKILL.md)        |
| `integration-testing`   | Patterns for integration tests (Jest)                  | [SKILL.md](.agent/skills/integration-testing/SKILL.md)   |
| `jsdoc`                 | Standards for JSDoc typing                             | [SKILL.md](.agent/skills/jsdoc/SKILL.md)                 |
| `web-design-guidelines` | Review UI code for Web Interface Guidelines compliance | [SKILL.md](.agent/skills/web-design-guidelines/SKILL.md) |

### Auto-invoke Skills

When performing these actions, ALWAYS invoke the corresponding skill FIRST:

| Action                                                 | Skill                   |
| ------------------------------------------------------ | ----------------------- |
| Create new AI agent skill                              | `skill-creator`         |
| Create or modify a view                                | `nunjucks-views`        |
| Create or modify integration tests                     | `integration-testing`   |
| Define entities, repos, or services                    | `jsdoc`                 |
| Review UI code for Web Interface Guidelines compliance | `web-design-guidelines` |

---

## Project Overview

`clinica-angel` is a university project for a Clinic Appointment Scheduler. It is a monolithic Node.js web application that uses Server-Side Rendering (SSR) for the frontend and exposes an internal API.

The project follows a **Modular Architecture** (Vertical Slicing), where each major feature (e.g., `auth`, `users`, `patients`) is self-contained with its own layers.

| Component | Location                 | Tech Stack                         |
| --------- | ------------------------ | ---------------------------------- |
| Backend   | `src/`                   | Node.js, Express, Prisma (MariaDB) |
| Frontend  | `src/{slice}/views/`     | Nunjucks, Tailwind CSS, DaisyUI    |
| Database  | `prisma/`                | Prisma Schema                      |
| Tests     | `src/**/*.{int}.test.js` | Jest                               |

---

## Architecture

### Directory Structure

The codebase is organized by **Feature Modules**:

- `src/_shared/`: Common code used across modules (Database connection, global errors, base views).
- `src/<feature>/`: (e.g., `src/users/`)
  - `domain/`: Pure business logic, entity models, and repository interfaces (defined via JSDoc). **No external dependencies** (except maybe utility libs).
  - `application/`: Service layer implementing use cases. Orchestrates domain objects and repositories.
  - `infrastructure/`: Framework-specific code. Database implementations (Prisma), HTTP controllers, Routers, DTO schemas.
  - `views/`: Nunjucks templates (`.njk`) for UI.

## Key Files

- `src/app.js`: Main Express application setup, middleware, and view engine configuration.
- `prisma/schema.prisma`: Database schema definition.
- `src/_shared/infrastructure/prisma.js`: Prisma client instance.
- `src/_shared/infrastructure/services-container.js`: Central dependency injection point. Instantiates repositories and services, wiring them together, and exports a `services` object for use in controllers.

## Node.js Development

```bash
# Setup
pnpm install
pnpm dlx prisma generate
pnpm dev

# Code quality
pnpm lint:fix
pnpm format
```

---

## Commit Guidelines

Follow conventional-commit style: `<type>[scope]: <description>`

**Types:** `feat`, `fix`, `docs`, `chore`, `perf`, `refactor`, `style`, `test`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EmanuelAngel)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/EmanuelAngel)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
