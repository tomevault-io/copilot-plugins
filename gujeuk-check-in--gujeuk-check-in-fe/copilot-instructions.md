## gujeuk-check-in-fe

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

GuJeuk Check-in is a visitor management system frontend for tracking visitor logs, managing user information, and customizing visit purposes. Built with React 19 and Vite (using rolldown-vite).

## Commands

```bash
yarn dev       # Start development server
yarn build     # Build for production
yarn preview   # Preview production build
```

Note: `yarn lint` currently just echoes "Lint Pass" - no actual linting runs.

## Architecture

This project follows **Feature-Sliced Design (FSD)** architecture with the following layers:

- **app/** - Application entry, router configuration, global providers (React Query, BrowserRouter)
- **pages/** - Route-level components (auth, visit, user, purpose pages)
- **widgets/** - Composite UI blocks (layouts, headers, forms)
- **features/** - User actions with business logic (login, CRUD operations for visits/users/purposes)
- **entities/** - Business entities with their APIs, types, and stores (auth, user, visit, purpose)
- **shared/** - Reusable utilities, hooks, UI components, and API instance

### Path Aliases

```
@app/*       -> src/app/*
@pages/*     -> src/pages/*
@widgets/*   -> src/widgets/*
@features/*  -> src/features/*
@entities/*  -> src/entities/*
@shared/*    -> src/shared/*
```

## Key Technical Details

- **State Management**: Zustand with persist middleware (localStorage for auth)
- **Data Fetching**: TanStack Query (React Query) + Axios
- **Styling**: Emotion (css-in-js with `@emotion/react` and `@emotion/styled`)
- **Routing**: React Router DOM v7
- **Drag & Drop**: @dnd-kit for sortable purpose ordering
- **Language**: Mixed JSX/TSX (gradually migrating to TypeScript)

### Authentication Flow

- JWT-based auth with access/refresh tokens stored in Zustand (persisted to localStorage)
- Axios interceptors handle automatic token refresh on 401 responses
- Auth store at `src/entities/auth/model/authstore.ts`
- Token refresh logic in `src/shared/api/axiosInstance.ts`

### API Configuration

- Base URL configured via `VITE_API_BASE_URL` environment variable
- All API calls go through the shared axios instance with auth interceptors

## Main Routes

- `/organ/login` - Admin login
- `/organ/change` - Change password
- `/log` - Visit log list
- `/log/create` - Create new visit log
- `/log/:logId` - View visit log detail
- `/organ/user/all` - User list
- `/organ/user/:userId` - User detail
- `/purpose/all` - Purpose customization

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GuJeuk-Check-in)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/GuJeuk-Check-in)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
