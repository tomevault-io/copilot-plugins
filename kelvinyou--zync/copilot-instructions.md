## zync

> - **Frontend**: React 18, TypeScript, Shadcn, Vite, React Query, React Hook Form


# Zync Architecture

## Tech Stack
- **Frontend**: React 18, TypeScript, Shadcn, Vite, React Query, React Hook Form
- **Backend**: NestJS, Supabase
- **Monorepo**: Nx workspace with yarn

## Package Structure

packages/
├── frontend/          # React frontend application
├── backend/           # NestJS frontend application
└── shared/            # Common types and utilities

## Key Principles
- **Functional components only** (no classes)
- **Named exports only** (no default exports)
- **Types over interfaces** (except for extending third-party)
- **String literals over enums** (except GraphQL)
- **No 'any' type allowed**
- **Event handlers over useEffect** for state updates

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KelvinYou) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
