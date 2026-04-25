## neurosama-management

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Robotics Team Manager is a TanStack Start web application for managing FTC (FIRST Tech Challenge) robotics team operations including competitions, tasks, robot subsystems, inventory, engineering notebooks, scouting, and meetings.

**Stack**: TanStack Start, TanStack Router, React 19, Convex (backend/database), Clerk (authentication), Tailwind CSS, Radix UI/shadcn, Vite

## Common Commands

```bash
npm run dev           # Start both frontend (Vite) and backend (Convex)
npm run dev:frontend  # Start only Vite dev server
npm run dev:backend   # Start only Convex backend
npm run build         # Production build
npm run lint          # Run ESLint
```

## Architecture

### Frontend (`src/app/`)
- Uses TanStack Router file-based routing
- `__root.tsx` - Root layout with providers (Clerk, Convex, Auth, Timezone)
- `_dashboard.tsx` - Layout route wrapping protected dashboard pages
- `_dashboard/` - Protected routes requiring authentication
  - `index.tsx` - Dashboard home page
  - `projects.tsx`, `projects.$id.tsx` - Projects and detail pages
  - `tasks.tsx` - My Tasks page
  - etc.
- `login.tsx`, `signup.tsx` - Clerk authentication pages
- Path alias: `@/*` maps to `./src/*`

### Backend (`src/convex/`)
- Convex handles database, real-time subscriptions, and server functions
- `schema.ts` - Database schema with all table definitions
- Each feature has its own file (e.g., `tasks.ts`, `projects.ts`, `competitions.ts`)
- `_generated/` - Auto-generated types (do not edit)

### Authentication Flow
1. Clerk handles sign-in/sign-up
2. `_dashboard.tsx` layout uses RequireAuth component to protect routes
3. `AuthContext` manages user state and approval status via Convex
4. User roles: "owner", "admin", "member"

### Data Patterns
```typescript
// Fetching data (auto-subscribes to real-time updates)
const data = useQuery(api.module.queryName, { args });

// Mutations
const mutate = useMutation(api.module.mutationName);
await mutate({ args });

// Auth context
const { user, isApproved, authStatus } = useAuthContext();

// Navigation with TanStack Router
import { Link, useNavigate } from "@tanstack/react-router";
<Link to="/projects/$id" params={{ id: projectId }}>View Project</Link>

const navigate = useNavigate();
navigate({ to: "/projects" });
```

### Key Directories
- `src/components/ui/` - Reusable shadcn/ui components
- `src/components/layout/` - AppShell, Sidebar
- `src/components/{feature}/` - Feature-specific components (projects/, tasks/, teams/, kanban/)
- `src/context/AuthContext.tsx` - Authentication state management
- `src/router.tsx` - TanStack Router configuration

## Database Schema Highlights

Core tables in `src/convex/schema.ts`:
- **users/waitlist** - User management with approval flow
- **teams/teamMembers** - Team entities and membership
- **projects/tasks** - Work management with assignments
- **competitions/competitionMatches** - FTC event tracking
- **robotSubsystems/partsInventory** - Robot and parts management
- **engineeringNotebook/scoutingReports** - Documentation
- **meetings/attendance** - Team operations

## Convex Guidelines

From `.cursor/rules/convex_rules.mdc`:
- Use new function syntax with explicit validators
- Reference functions via `api.module.function` or `internal.module.function`
- Queries should be indexed for performance

## Clerk Guidelines

- Use `@clerk/clerk-react` for TanStack Start (not `@clerk/nextjs`)
- ClerkProvider wraps the app in root layout (`__root.tsx`)
- Use `useAuth`, `SignInButton`, `SignUpButton` from `@clerk/clerk-react`

## Environment Variables

Required:
- `VITE_CONVEX_URL` - Convex deployment URL
- `VITE_CLERK_PUBLISHABLE_KEY` - Clerk publishable key

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Webb-exe) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
