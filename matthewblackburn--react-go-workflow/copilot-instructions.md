## react-go-workflow

> - **Frontend**: React, TypeScript, Vite, Tailwind CSS, shadcn/ui (radix), React Flow (@xyflow/react)

# Project: React + Go Workflow Builder

## Stack
- **Frontend**: React, TypeScript, Vite, Tailwind CSS, shadcn/ui (radix), React Flow (@xyflow/react)
- **Backend**: Go, ent ORM, Chi router
- **Tooling**: pnpm (web), Biome (lint/format), Vitest (tests)

## Frontend Feature Structure

```
web/src/features/<feature>/
├── components/    # components used by pages in this feature
├── utils/         # utility functions for this feature
├── pages/
│   ├── ViewAll.tsx
│   ├── ViewOne.tsx
│   └── EditOne.tsx
├── api.ts         # API calls for this feature
├── hooks.ts       # custom hooks for this feature
└── types.ts       # TypeScript types for this feature
```

Rules:
- Flat structure — components, utils, pages are siblings at the feature root
- Pages are plain files inside `pages/`, no subfolders
- Feature-level files (`api.ts`, `hooks.ts`, `types.ts`) live at the feature root

## Commands
- `cd web && pnpm run typecheck` — TypeScript check
- `cd web && pnpm run lint` — Biome lint
- `cd web && npx vitest run` — Run tests
- `docker compose up -d --build --renew-anon-volumes web` — Rebuild web container

---
> Source: [matthewblackburn/react-go-workflow](https://github.com/matthewblackburn/react-go-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
