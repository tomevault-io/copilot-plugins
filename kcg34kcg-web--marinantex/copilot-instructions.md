## marinantex

> You are an expert Senior Full Stack Architect and AI Engineer. Your task is to build the foundation of "Babylexit," a next-generation AI-powered Legal Operating System (SaaS) for law firms in Turkey.

# .cursorrules

# Role & Objective
You are an expert Senior Full Stack Architect and AI Engineer. Your task is to build the foundation of "Babylexit," a next-generation AI-powered Legal Operating System (SaaS) for law firms in Turkey.

This system features:
1.  **Digital Twin:** An AI agent for lawyers (RAG-based).
2.  **Glass Window:** A transparent client portal.

# Tech Stack & Standards
-   **Framework:** Next.js 15 (App Router, Server Components, Server Actions).
-   **Language:** TypeScript (Strict Mode, no `any`).
-   **Auth & DB:** Supabase (Auth, PostgreSQL, Vector, Realtime).
-   **Package:** `@supabase/ssr` (strictly for cookies handling).
-   **Styling:** Tailwind CSS + Shadcn/UI.
-   **Design System:**
    -   **Background:** Clean White (`#FFFFFF`) as the dominant base.
    -   **Primary:** Professional Blue (Trust/Legal) - e.g., `blue-600` for primary buttons/headers.
    -   **Accent:** Vibrant Orange (Action/Attention) - e.g., `orange-500` for CTAs and highlights.
    -   **Typography:** Inter or Geist Sans. Clean, high contrast.
-   **Icons:** Lucide React.
    -   **Legal Context:** Use distinct icons for legal concepts (e.g., `Scale` for justice, `Gavel` for court/rulings, `FileText` for docs, `ShieldAlert` for urgency, `Scroll` for statutes).
-   **Localization & Formatting (CRITICAL):**
    -   **Language:** All UI text, labels, and user-facing content must be in **Turkish**.
    -   **Date Format:** STRICTLY `DD.MM.YYYY` (e.g., 25.04.2026). Never use MM/DD/YYYY.
    -   **Library:** Use `date-fns` with `tr` locale for all date rendering and manipulation.
-   **State Management:**
    -   **Server State:** React Query (TanStack Query) - STRICTLY for data fetching/caching.
    -   **Client State:** Zustand - For UI state (sidebar, modals).
-   **AI:** Vercel AI SDK (Core & UI) via `api/chat`.
-   **Validation:** Zod (Schema validation for Forms and APIs).

# Core Architecture & Roles
1.  **Lawyer (Admin):** Full CRUD access (`/dashboard`, `/cases`, `/clients`).
2.  **Client (Viewer):** Restricted access (`/portal`). Can only view explicitly shared resources (RLS protected).

# Implementation Guidelines

## 1. Project Setup & Configuration
-   Initialize Next.js with TypeScript and Tailwind.
-   **Theme Configuration:** Configure `tailwind.config.ts` and `globals.css` to use CSS variables.
    -   Base: White.
    -   Ring/Focus/Accent: Orange.
    -   Primary Foreground: Blue.
-   **Route Groups:** Use strict separation:
    -   `(auth)`: Login/Signup pages.
    -   `(dashboard)`: Lawyer interface (Sidebar layout).
    -   `(portal)`: Client interface (Simplified layout).
-   **Middleware:** Create `middleware.ts` using `@supabase/ssr` `createServerClient` to handle session refreshing and route protection.
    -   *Logic:* If no session -> redirect to `/login`. If session exists but wrong role -> 403 or redirect.

## 2. Database Schema (Supabase SQL)
Write a robust `schema.sql` (to be executed in SQL Editor). Must include:
-   **Extensions:** `vector` (for embeddings), `uuid-ossp`.
-   **Enums:** `user_role` ('lawyer', 'client').
-   **Tables:**
    -   `profiles`: (PK `id` references `auth.users`, `full_name`, `role`, `avatar_url`).
    -   `cases`: (PK `id`, `title`, `status`, `lawyer_id` references profiles, `created_at` timestamptz).
    -   `documents`: (PK `id`, `case_id`, `content`, `embedding` vector(1536), `file_path`).
    -   `case_updates`: (PK `id`, `case_id`, `message`, `date`, `is_public_to_client` BOOLEAN).
    -   `ai_chats`: (PK `id`, `user_id`, `messages` jsonb).
-   **RLS Policies (CRITICAL):**
    -   Enable RLS on all tables.
    -   **Lawyers:** `USING (auth.uid() = lawyer_id)` for Cases. Full CRUD.
    -   **Clients:** `USING (auth.uid() = client_id)` for Cases (SELECT only).
    -   **Updates:** Clients can only SELECT `case_updates` where `is_public_to_client = true`.

## 3. The Dashboard (Lawyer View - `/dashboard`)
-   **Layout:** Collapsible Sidebar (Zustand) + Header (Breadcrumbs, UserMenu).
-   **Page:** `/dashboard/page.tsx`
    -   "Morning Briefing" Card (AI Summary placeholder in Turkish).
    -   "Upcoming Deadlines" (Mock data, properly typed Dates in `DD.MM.YYYY`).
    -   Use **Shadcn Skeleton** for loading states.

## 4. Case Management (`/dashboard/cases`)
-   **List View:** TanStack Table (Shadcn `DataTable`).
    -   Columns: Case Name (Dosya Adı), Client (Müvekkil), Status (Badge: Blue/Orange), Last Updated (Son Güncelleme - DD.MM.YYYY).
    -   Filters: Status, Client Name.
-   **Detail View:** `/dashboard/cases/[id]/page.tsx`
    -   **Tabs:** Overview (Genel Bakış), Documents (Belgeler - Drag & Drop), Timeline (Zaman Çizelgesi).
    -   **Timeline:** Visual distinction: Blue border for "Internal" (Dahili), Orange border for "Client Visible" (Müvekkil Görür).

## 5. "Digital Twin" AI Chat
-   **Component:** `components/ai-chat.tsx`.
    -   Floating or Sidebar panel.
    -   Streaming text UI (use `useChat` from `ai/react`).
-   **Backend:** `app/api/chat/route.ts`.
    -   Implement `streamText` from `ai`.
    -   *Mock:* For Phase 1, return a streamed mock response to save credits, but setup the context retrieval structure.

# Anti-Bug & Stability Rules
1.  **Hydration:** Always use `suppressHydrationWarning` on the `html` tag or specific date components to avoid mismatches due to timezone differences.
2.  **Type Safety:** Define strict interfaces in `types/database.ts` (generated from Supabase) and `types/index.ts`. Never use `any`.
3.  **Server Actions:**
    -   Use `try/catch` in all actions.
    -   Return standardized objects: `{ success: boolean, data?: T, error?: string }`.
    -   Use `zod` to validate inputs *before* sending to DB.
4.  **Environment:** Access variables strictly via `process.env`. Fail fast if `NEXT_PUBLIC_SUPABASE_URL` or `NEXT_PUBLIC_SUPABASE_ANON_KEY` are missing.

# Immediate Action Plan
Please execute the following steps in order:
1.  **Generate SQL Schema:** Output the complete SQL code for Supabase (Tables + RLS + Vector).
2.  **Project Structure:** Output the file tree showing the (auth), (dashboard), and (portal) separation.
3.  **Type Definitions:** Create `types/index.ts` covering Profile, Case, and Document interfaces.
4.  **Auth Utility:** Write the `utils/supabase/server.ts` and `middleware.ts` code using `@supabase/ssr`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kcg34kcg-web) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
