## mono-repo-initializer-nest-dashboard-nuxt

> This is a pnpm monorepo with NestJS backend, Nuxt dashboard (Shadcn), and Nuxt frontend (Nuxt UI).

# Monorepo Project Rules

This is a pnpm monorepo with NestJS backend, Nuxt dashboard (Shadcn), and Nuxt frontend (Nuxt UI).

## Package Manager

Always use `pnpm`. Never use npm or yarn.

## Workspace Apps

### Backend (apps/backend/)

- NestJS 11 with Prisma 7
- PostgreSQL database
- RESTful API at /api prefix
- Modular architecture in src/modules/

### Dashboard (apps/dashboard/)

- Nuxt 3 with Shadcn-vue components
- TailwindCSS with CSS variables
- Components in app/components/ui/

### Frontend (apps/frontend/)

- Nuxt 3 with Nuxt UI
- Nuxt Content for Markdown
- Content in content/ directory

## Key Patterns

### NestJS Modules

```
src/modules/{feature}/
├── dto/
├── {feature}.controller.ts
├── {feature}.service.ts
├── {feature}.module.ts
└── index.ts
```

### Vue Components

Use Composition API with `<script setup lang="ts">`.

### Prisma

Schema at `apps/backend/prisma/schema.prisma`. Run `pnpm prisma:generate` after schema changes.

## Documentation

- Root: AGENTS.md
- Backend: apps/backend/AGENTS.md, apps/backend/docs/
- Dashboard: apps/dashboard/AGENTS.md
- Frontend: apps/frontend/AGENTS.md

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AnwarKXC) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
