## studio

> **1. Project Overview:**

# Project Status Summary

**1. Project Overview:**
*   Next.js application with TypeScript.
*   Styling: Tailwind CSS.
*   Backend: Supabase (used for database and authentication).
*   Form Management: React Hook Form, Zod.
*   Charting: Recharts.

**2. Initial Setup & Environment Challenges:**
*   Encountered persistent `npm` cache permission issues (`EACCES`) preventing global package installations (e.g., `eslint`, `yarn`) and `npm run lint` from completing successfully. Attempts to clear cache or change ownership via `sudo` were unsuccessful due to non-interactive terminal limitations.
*   ESLint was initially unconfigured; a `.eslintrc.json` with `next/core-web-vitals` was manually created.
*   The application's development server (`npm run dev`) was found to be already running on port 9002.

**3. Completed Refactoring & Bug Fixes:**

*   **`src/context/DataContext.tsx`:**
    *   Implemented `loading` and `error` states for data fetching.
    *   Refactored data fetching to occur in parallel using `Promise.all` for initial load.
    *   Memoized the `DataContext` provider `value` to prevent unnecessary re-renders of consuming components.
    *   Updated `makeApiRequest` and `deleteCleaningTaskDefinition` to trigger a full data re-fetch after mutations, ensuring data freshness.
*   **API Routes (`src/app/api/**/route.ts`):**
    *   **Security & Efficiency:** Replaced direct, insecure `createClient` calls using `SUPABASE_SERVICE_ROLE_KEY` with new helper functions (`createSupabaseServerClient` for authenticated user access and `createSupabaseAdminServerClient` for admin-level operations).
    *   **Authentication/Authorization:** Applied `withAuth` and `withAdminAuth` middleware consistently across all API routes to enforce proper access control (e.g., `POST`, `PUT`, `DELETE` operations now generally require admin privileges).
    *   **Data Mapping:** Removed manual snake_case to camelCase data mapping in API responses and requests, relying on Supabase's automatic handling where possible.
    *   **Hardcoded Logic:** In `temperature-logs` routes, replaced hardcoded system parameters with dynamic fetching from the `system_parameters` table in Supabase.
    *   **File Structure:** Created `src/lib/supabase/server.ts` to centralize server-side Supabase client creation logic.
    *   **Cleanup:** Removed the deprecated `src/lib/supabase.ts` file.
*   **`src/lib/auth-middleware.ts`:**
    *   Updated to utilize the new server-side Supabase client functions.
    *   Resolved duplicate `NextRequest` and `NextResponse` import declarations.
*   **`src/lib/types.ts`:**
    *   Updated `DataContextType` interface to include the new `loading` and `error` properties.
*   **`src/app/page.tsx`:**
    *   Modified to consume and display the `loading` and `error` states from the `DataContext`.
*   **UI Components (`src/components/layout/main-nav.tsx`, `src/components/auth/PasswordChangeForm.tsx`):**
    *   Corrected the rendering of the `PasswordChangeForm` within the `DropdownMenu` to properly use `DialogTrigger` and `children` props, resolving a UI rendering bug.

**4. Current State & Next Steps:**

*   **Type Checking:** `npm run typecheck` currently passes with no errors.
*   **Linting:** `npm run lint` is still failing due to the underlying `npm` cache permission issues, preventing a full linting pass. This is a system-level environment problem.
*   **Functional Verification:** No functional tests have been run yet. The next critical step is to verify the application's functionality, especially given the user's report of it being "very buggy and basically not working." This will likely involve running the development server and manually testing features.
*   **Comprehensive Code Review:** While significant refactoring has occurred, a deeper review of individual page components, custom hooks, and utility functions is still needed to identify and address any remaining bugs, inefficiencies, or non-idiomatic patterns.
*   **Database Schema:** The refactoring assumes the Supabase database schema aligns with the application's data models. This should be verified.
*   **Error Handling (Client-side):** While `DataContext` now exposes errors, individual components might need more granular error handling and display.
*   **UI/UX:** The user's initial description suggests broader UI/UX issues that will need to be addressed once core functionality is stable.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/futr3t)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/futr3t)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
