## accounting-manager

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md — accounting-manager

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

Dashboard frontend for small business/shop accounting. Main workflow: daily cash register closing (income entry) + expense tracking, with visibility into how the accounting is going.

---

## Stack

| Area | Technology |
|---|---|
| Framework | Next.js 15 (App Router + Turbopack) |
| Language | TypeScript 5 |
| Styles | Tailwind CSS 4 + CSS custom properties |
| State | Redux Toolkit (react-redux) |
| Charts | Chart.js + react-chartjs-2 |
| HTTP | Axios |
| Validation | Zod |
| UI Components | Headless UI, Lucide React, Heroicons |
| Testing | Jest + React Testing Library |

---

## Architecture

### API Proxy Pattern
Next.js API routes in `src/app/api/` proxy requests to the NestJS backend at `localhost:3000`. Cookies are forwarded for JWT authentication.

```
React component -> fetch('/api/...') -> Next.js API route -> axios('http://localhost:3000/...') -> NestJS
```

### Redux Store
Located at `src/app/store.ts`. Current slices:
- `user` — authentication state (`src/features/user/userSlice`)
- `book` — accounting book selection (`src/features/book/bookSlice`)

Types: `RootState`, `AppDispatch` exported from store.

### Design System
CSS tokens defined in `src/app/globals.css`:
```css
--color-dashboard-bg: #F8F9FB;
--color-dashboard-sidebar: #FFFFFF;
--color-dashboard-accent: #2B616D;    /* teal — primary action color */
--color-dashboard-text: #1A1D1F;
--color-dashboard-secondary: #6F767E;
--color-dashboard-border: #F1F1F1;
```

Use Tailwind classes: `bg-dashboard-bg`, `text-dashboard-accent`, etc. Never hardcode hex colors in components.

---

## Folder Structure

```
src/
├── app/
│   ├── api/            # Next.js API routes (proxy to backend)
│   │   ├── auth/
│   │   ├── book/ books/
│   │   ├── budgets/
│   │   ├── categories/
│   │   ├── daily-reports/
│   │   ├── periodic/
│   │   ├── suppliers/
│   │   ├── transactions/
│   │   └── users/
│   ├── components/     # Shared UI components
│   │   ├── AppProviders/
│   │   ├── AuthInitializer/
│   │   ├── ConditionalNav/
│   │   ├── Form/
│   │   ├── LoginModal/
│   │   ├── Modal/
│   │   └── Nav/
│   ├── Dashboard/      # Main dashboard page
│   ├── categories/     # Categories page
│   ├── suppliers/      # Suppliers page
│   ├── transactions/   # Transactions page
│   ├── login/          # Login page
│   ├── context/        # React contexts
│   ├── hooks/          # Custom hooks
│   ├── lib/            # Utilities
│   └── types/          # TypeScript types
├── features/           # Redux slices
│   ├── user/userSlice
│   └── book/bookSlice
```

---

## Code Conventions

- `'use client'` for components with state, effects, or browser APIs
- Props typed with `interface`, not inline `type`
- Tailwind utility classes with dashboard-* tokens
- Icons from `lucide-react` or `@heroicons/react`
- API routes forward cookies: `headers: { Cookie: req.headers.get('cookie') }`
- Zod schemas for form validation

---

## Commands

```bash
npm run dev          # Start at localhost:3001 (Turbopack)
npm run build        # Production build
npm run lint         # ESLint
npm test             # Jest tests
npm run test:watch   # Watch mode
npm run test:cov     # Coverage report
npx tsc --noEmit     # Type-check
```

---

## Skills (slash commands)

| Skill | When to invoke |
|---|---|
| `/new-component` | Create a new UI component |
| `/new-page` | Create a new Next.js page |
| `/new-api-route` | Create a new API proxy route |
| `/new-chart` | Create a Chart.js component |
| `/redesign-page` | Redesign an existing page |
| `/project-status` | Check project status and missing functionality |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jjpg-labs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
