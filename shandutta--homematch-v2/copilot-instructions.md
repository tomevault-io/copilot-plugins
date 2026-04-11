## homematch-v2

> > **Role**: This file serves as the primary context source for AI agents (Gemini, Claude, Copilot) working on the HomeMatch V2 codebase. It synthesizes architecture, patterns, and domain knowledge.

# HomeMatch V2 - Gemini Context File

> **Role**: This file serves as the primary context source for AI agents (Gemini, Claude, Copilot) working on the HomeMatch V2 codebase. It synthesizes architecture, patterns, and domain knowledge.

## 1. Project Identity

- **Name**: HomeMatch V2
- **Core Value Prop**: "Tinder for Homes" with a "Multiplayer Mode" (Couples/Roommates).
- **Stage**: Migration from V1 complete. Foundation laid. Service layer ready. UI implementation in progress.

## 2. Technology Stack (Strict Adherence)

| Layer        | Technology                     | Key Details                                                       |
| ------------ | ------------------------------ | ----------------------------------------------------------------- |
| **Frontend** | **Next.js 15.4** (App Router)  | React 19, Server Components, Edge Runtime.                        |
| **Styling**  | **Tailwind CSS 4**             | + `shadcn/ui` (Radix primitives). Design tokens in CSS variables. |
| **Language** | **TypeScript 5.x**             | Strict mode. **Zod** for runtime validation everywhere.           |
| **Backend**  | **Supabase**                   | PostgreSQL 15+, GoTrue (Auth), Realtime, Edge Functions.          |
| **Database** | **PostgreSQL + PostGIS**       | Spatial queries (distance, bounds). RLS policies active.          |
| **State**    | **TanStack Query v5**          | Server state (caching, optimistic updates).                       |
| **State**    | **Zustand**                    | Client-only UI state (modals, swipe queues).                      |
| **Testing**  | **Jest / Vitest / Playwright** | 100% Unit/Integration pass rate required.                         |

## 3. Architecture & Patterns

### A. Data Flow

1.  **Client**: `useQuery` (TanStack) calls API Route or Server Action.
2.  **API Route**: Validates input via **Zod Schema**. Calls `Service Layer`.
3.  **Service Layer**: (e.g., `PropertyService`) contains business logic. Calls `Supabase Client`.
4.  **Database**: Postgres executes query + RLS policies + PostGIS functions.

### B. Key Domain Models

- **Property**: The core entity. Sourced from Zillow (ZPID). Includes `images`, `price`, `coordinates` (PostGIS Point).
- **Household**: A group of 2+ users.
  - _Logic_: Interactions are scored. If Partner A "likes" and Partner B "likes", it's a **Match**.
- **Interaction**: `view`, `like`, `dislike`, `skip`. Unique per `(user_id, property_id)`.
- **Neighborhood**: Geographic polygon. Used for aggregate stats (Walk Score, Safety).

### C. Directory Structure Map

```text
/src
  /app              # Next.js App Router (File-system routing)
    /api            # API Routes (strictly typed)
    /auth           # Auth pages & callbacks
    /dashboard      # User Dashboard
  /components
    /ui             # Generic shadcn components (Button, Card)
    /features       # Domain specific (PropertyCard, LoginForm)
  /lib
    /supabase       # Clients (Browser, Server, ServiceRole)
    /services       # Business Logic (PropertyService.ts, UserService.ts)
    /schemas        # Zod Schemas (Single source of truth for types)
  /hooks            # Custom React Hooks (useAuth, useProperties)
  /types            # TS Interfaces (often inferred from Zod)
/supabase
  /migrations       # SQL DDL files (Single source of truth for DB)
/tests              # Testing roots
```

## 4. Development Guidelines

### Coding Standards

- **Type Safety**: Never use `any`. Use Zod to validate API inputs/outputs.
- **Components**: Functional components only. Use `hooks` for logic extraction.
- **Styling**: Tailwind classes. No CSS files modules unless absolutely necessary.
- **Async**: Use `async/await`. Handle errors with `try/catch` in API routes.

### Testing Strategy

- **Unit (Jest)**: For utilities, hooks, and isolated components.
- **Integration (Vitest)**: For Service Layer and API Routes (uses local Supabase).
- **E2E (Playwright)**: For critical user flows (Signup, Swiping).

### Common Commands

- `pnpm dev`: Start dev server.
- `pnpm test`: Run all tests.
- `pnpm test:unit`: Run Jest.
- `pnpm test:integration`: Run Vitest.
- `pnpm test:e2e`: Run Playwright.
- `supabase status`: Check local backend.

## 5. Current Status (as of Nov 2025)

- **Completed**:
  - DB Migration (Tables + RLS + PostGIS).
  - Auth Flow (Login/Signup + Google OAuth).
  - Service Layer (Properties, Users, Interactions).
  - CI/CD Pipeline (GitHub Actions).
- **In Progress**:
  - Frontend UI implementation (Dashboard, Swipe Interface).
  - Integration of `PropertySwiper` component.
- **Pending**:
  - ML Scoring System integration.
  - Advanced Notifications.

## 6. Critical Contexts

- **Google Maps**: Uses a **Secure Proxy** (`src/app/api/maps/...`). Never expose API keys on client.
- **Images**: Stored in Supabase Storage or external URLs (Zillow). Handled via Next.js `<Image/>` optimization.
- **RLS**: Row-Level Security is **strict**. Users can only see their own data or their household's data.

---

_This file is auto-maintained. Read `docs/` for deep dives._

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shandutta)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/shandutta)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
