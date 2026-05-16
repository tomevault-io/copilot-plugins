## web-daycare

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev        # Start development server (Refine dev)
npm run build      # TypeScript compile + Refine build
npm run start      # Start production server
```

No lint or test scripts are configured in package.json.

## Architecture Overview

This is a **Daycare Management System** built with **React 19 + Refine.dev 5 + Ant Design 5**. It manages users, children, attendance, incidents, documents, and calendar events.

### Key directories

```
src/
├── pages/          # Route-level components (UI only, consume domains/hooks)
├── domains/        # Domain logic per entity: api/, types/, utils/, index.ts
├── shared/
│   ├── config/     # axios.config.ts (HTTP client), app.config.ts (routes & resources)
│   ├── hooks/      # useAuth, useData, usePermissions, useNavigation, useRefineHooks
│   ├── components/ # Layout, sidebar, shared UI components
│   ├── providers/  # Custom notification provider
│   ├── i18n/       # locale.ts — English & Spanish with dayjs/moment/Ant Design locales
│   └── routes/     # app.routes.tsx — main router config
├── App.tsx         # Refine setup, providers, i18n
├── authProvider-refine-native.ts   # Refine auth provider (JWT)
└── dataProvider-stable-fixed.ts    # Refine data provider with 30s in-memory cache
```

### Data flow

- **Pages** call **domain APIs** or **shared hooks** (`useData`, `useAuth`)
- **Domain APIs** use the shared Axios instance from `shared/config/axios.config.ts`
- **Refine data provider** (`dataProvider-stable-fixed.ts`) wraps API calls with 30s caching, automatic pagination (`take` max 150), and cache invalidation on mutations
- React Query is configured: `staleTime: 30s`, `gcTime: 5min`, `retryCount: 0`

### Authentication

- JWT stored in `localStorage` under key `refine-auth`; user data under `user`
- Axios request interceptor attaches `Authorization: Bearer <token>` to all requests
- Response interceptor handles 401 → auto logout; parses NestJS error formats for human-readable messages
- Auth provider in `authProvider-refine-native.ts` implements Refine's `login`, `logout`, `check`, `getPermissions`, `getIdentity`, `onError`
- Roles: `Administrator`, `Educator`, `Parent`

### API

- Base URL: `https://api.thechildrenworld.com/api` (can be overridden to `http://localhost:30002/api`)
- Special endpoint conventions:
  - Attendance sub-resources use `/all` suffix
  - Children detail uses `with-relations` suffix

### Internationalization

- Language stored in `localStorage` under `app-language`; default English
- Two locales: `en-US`, `es-CO`
- Translations are inline objects in component files (e.g., `LOGIN_TRANSLATIONS`, `AUTH_TRANSLATIONS`) — not external JSON files
- App resource labels (sidebar menu) are translated in `shared/config/app.config.ts`
- Ant Design, dayjs, and moment locales all switch together via `shared/contexts/language.context.tsx`

### Domain pattern

Each domain follows this structure:
```
domains/<entity>/
├── api/       # Axios-based API calls
├── types/     # TypeScript interfaces
├── utils/     # Helper functions
└── index.ts   # Re-exports
```

### TypeScript paths

`tsconfig.json` sets `baseUrl: "src"`, so imports use bare paths like `shared/hooks/use-auth.hook` instead of relative `../../` paths.

---
> Source: [david0ql/web-daycare](https://github.com/david0ql/web-daycare) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
