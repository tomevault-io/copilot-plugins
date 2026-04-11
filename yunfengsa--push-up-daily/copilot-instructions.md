## push-up-daily

> This is a Next.js (v16.1.2) application designed for the Chinese market ("push-up-daily"), featuring a secure authentication system built with `better-auth`, a PostgreSQL database, and a comprehensive statistics dashboard for tracking push-up progress over time. The UI is styled with Tailwind CSS v4 and is fully localized in Chinese.

# Project Overview

This is a Next.js (v16.1.2) application designed for the Chinese market ("push-up-daily"), featuring a secure authentication system built with `better-auth`, a PostgreSQL database, and a comprehensive statistics dashboard for tracking push-up progress over time. The UI is styled with Tailwind CSS v4 and is fully localized in Chinese.

## Key Technologies

*   **Framework:** Next.js 16.1.2 (App Router)
*   **Authentication:** `better-auth` (v1.4.13)
    *   **Plugins:** `username` (enables username/password login alongside email)
    *   **Database:** `pg` (PostgreSQL connection pool)
*   **Styling:** Tailwind CSS v4
*   **Visualization:** `recharts` (Charts & Graphs)
*   **Testing:** Jest, React Testing Library
*   **PWA:** Manifest configuration for "每日俯卧撑"
*   **Language:** TypeScript
*   **Routing Guard:** Next.js Middleware (renamed to `proxy.ts` per Next.js 16 conventions)

## Architecture

*   **`lib/db.ts`**: Centralized PostgreSQL connection pool using `pg`. Shared across business logic.
*   **`app/auth.ts`**: Server-side Better Auth configuration. Uses the shared pool from `lib/db.ts`.
*   **`app/api/auth/[...all]/route.ts`**: Exposes Better Auth endpoints.
*   **`lib/auth-client.ts`**: Client-side Better Auth instance.
*   **`proxy.ts`**: Handles route protection. Redirects unauthenticated users to `/auth`.
*   **`app/page.tsx`**: Entry point that redirects authenticated users to the home dashboard.
*   **`app/home/`**: 
    *   `page.tsx`: Push-up tracking dashboard with timer and manual entry.
    *   `actions.ts`: Server actions for persisting workout sessions.
*   **`app/summary/`**:
    *   `page.tsx`: Statistical dashboard with calendar view and date picker.
    *   `actions.ts`: Server actions for fetching and aggregating monthly performance data.
*   **`components/`**:
    *   `ui/`: Shared design system components (Button, Card, Modal, Skeleton, etc.).
    *   `pushup-chart.tsx`: Activity area chart using Recharts.
    *   `heatmap.tsx`: Contribution-style activity heatmap.
    *   `layout/`: Layout components like PageHeader.
*   **`scripts/init-pushup-db.sql`**: Database schema for the tracking feature.

## Building and Running

### Prerequisites
*   Node.js (v20+ recommended)
*   PostgreSQL Database (configured in `.env.local`)

### Commands

*   **Development Server:** `npm run dev`
*   **Production Build:** `npm run build`
*   **Linting:** `npm run lint`
*   **Testing:** `npm run test`
*   **Database Setup:**
    1. Better Auth tables: `npm run db:migrate` (or `npx @better-auth/cli migrate`)
    2. Business tables: `psql "$DATABASE_URL" -f scripts/init-pushup-db.sql`

## Data Schema

### `pushup_sessions`
| Field | Type | Description |
|---|---|---|
| `id` | INT | Primary Key |
| `user_id` | VARCHAR | Link to User table |
| `count` | INT | Number of push-ups |
| `duration` | INT | Time taken (seconds) |
| `created_at` | TIMESTAMP | Record timestamp |

## Development Conventions

*   **Localization:** All user-facing text must be in Chinese.
*   **Database:** Always use the shared pool from `@/lib/db`.
*   **Server Actions:** Use server actions for data mutations to leverage Next.js's security and type safety.
*   **Tailwind v4:** Adhere to v4 utility-first styling.
*   **UI Consistency:** Prioritize using components from `@/components/ui` to maintain the comprehensive design system (Glassmorphism, vibrant colors).

## constraints
后续的所有对话使用中文进行。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yunfengsa)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yunfengsa)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
