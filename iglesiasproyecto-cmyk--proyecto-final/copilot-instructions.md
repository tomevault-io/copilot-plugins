## proyecto-final

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # Start development server (Vite, hot-reload)
npm run build    # Build production bundle
npm install      # Install dependencies
```

No test or lint commands are configured.

## Architecture

**IGLESIABD** is a church management SPA built with React 18 + Vite + Tailwind CSS v4.

### Tech Stack
- **Routing**: React Router v7 with `routes.ts` defining 20+ routes
- **State**: Single global context in `src/app/store/AppContext.tsx` (~6k lines) — holds all app state with mock data and TypeScript interfaces matching the Supabase schema
- **UI**: shadcn/ui (Radix primitives) + MUI icons, with custom theme variables in `src/styles/theme.css`
- **Charts**: Recharts; **Animations**: Framer Motion; **Forms**: React Hook Form; **DnD**: React DnD
- **Styling**: Tailwind CSS v4 via `@tailwindcss/vite` plugin; path alias `@` → `src/`

### App Shell
```
main.tsx → App.tsx (RouterProvider) → RootLayout (AppProvider + ErrorBoundary)
  ├─ LandingPage / LoginPage  (public)
  └─ AppLayout (sidebar + nav) → feature pages (private)
```

### Data Layer (current state)
All data currently lives in `AppContext.tsx` as mock state. The app is **ready for Supabase integration** — the full schema is documented in `IGLESIABD_Supabase_Agent.md` (24 tables, 35 FKs across 6 domains: Geography, Churches/Leadership, Ministries, Users/Permissions, Events/Tasks, Academic Courses).

The backend integration roadmap is in `guidelines/Backend_Implementation_Plan.md` (6 phases: schema governance → read fetchers → mutations → auth → observability → CI/CD).

### Domain Model (Supabase schema)
- **Geography**: `pais → departamento → ciudad`
- **Churches**: `iglesia`, `pastor`, `sede` with junction tables
- **Ministries**: `ministerio`, `miembro_ministerio`
- **Users**: `usuario`, `rol`, `usuario_rol`, `notificacion`
- **Events/Tasks**: `tipo_evento`, `evento`, `tarea`, `tarea_asignada`
- **Academic**: `curso`, `modulo`, `evaluacion`, `proceso_asignado_curso`

### Design Tokens
Brand colors (defined in `src/styles/theme.css` as CSS custom properties):
- Primary: `#1a7fa8` (teal blue)
- Foreground/Navy: `#0c2340`
- Background/Cream: `#f5efe6`
- Supports light/dark mode via CSS variables

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/iglesiasproyecto-cmyk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-17 -->
