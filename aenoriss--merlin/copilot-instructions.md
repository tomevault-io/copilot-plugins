## merlin

> When creating or modifying any UI components, ALWAYS follow the design system skill at `.claude/skills/design-system/`.

# Project Guidelines

## Design System (IMPORTANT)
When creating or modifying any UI components, ALWAYS follow the design system skill at `.claude/skills/design-system/`.

Key requirements:
- **Language**: All UI text in Argentinean Spanish (use "vos" conjugation)
- **Colors**: Use CSS variables (bg-brand, bg-success, bg-warning, bg-destructive)
- **Components**: Use shadcn/ui
- **Icons**: Use Lucide React
- **Font**: Quicksand

## Project Structure
```
newProject/
├── frontend/          # Next.js + TypeScript + Tailwind + shadcn/ui + React Query
│   └── src/
│       ├── app/       # Next.js app router pages
│       ├── components/ui/  # shadcn/ui components
│       └── providers/ # React context providers
│
└── backend/           # Bun.js + Supabase
    └── src/lib/       # Supabase client
```

## Commands
- Frontend: `cd frontend && npm run dev` (port 3000)
- Backend: `cd backend && bun dev` (port 3001)

## Tech Stack
- **Frontend**: Next.js 16, React 19, TypeScript, Tailwind CSS v4, shadcn/ui, React Query, next-themes
- **Backend**: Bun.js, Supabase

---
> Source: [aenoriss/merlin](https://github.com/aenoriss/merlin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
