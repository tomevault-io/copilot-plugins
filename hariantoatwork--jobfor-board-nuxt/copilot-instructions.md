## jobfor-board-nuxt

> Project structure and architecture guidelines


# Job Application Organiser - Project Structure

This is a **Nuxt 4** project with **Vue 3** and **TypeScript** for a job application management system with Kanban-style boards.

## Core Architecture

### Main Entry Points
- [nuxt.config.ts](mdc:nuxt.config.ts) - Main Nuxt configuration
- [app/app.vue](mdc:app/app.vue) - Root application component
- [package.json](mdc:package.json) - Dependencies and scripts

### Key Directories
- `app/components/` - Vue components (Board/, UI/)
- `app/composables/` - Vue composables for state management
- `app/pages/` - File-based routing pages
- `app/layouts/` - Page layouts
- `app/types/` - TypeScript type definitions
- `app/utils/` - Utility functions
- `modules/` - Nuxt modules (auth, boards)
- `server/api/` - Server-side API routes
- `database/` - Database migrations and seeds

### Technology Stack
- **Nuxt 4** with Vue 3 Composition API
- **TypeScript** for type safety
- **Tailwind CSS** for styling
- **Better Auth** for authentication
- **Knex.js** for database queries
- **SQLite/PostgreSQL** for data storage
- **pnpm** as package manager (NOT npm)

## Development Rules
- Always use **ESM syntax** (`import`/`export`)
- Use **Composition API** with `<script setup>`
- Follow **en-UK** spelling conventions
- Use **pnpm** instead of npm for all package management
- Update [CHANGELOG.md](mdc:CHANGELOG.md) for all changes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HariantoAtWork) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
