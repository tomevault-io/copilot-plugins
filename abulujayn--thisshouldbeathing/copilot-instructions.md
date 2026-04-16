## thisshouldbeathing

> "This should be a thing" is a modern, multi-tenant Idea Board application. It allows users to propose ideas, vote, and discuss. It features a secure, passwordless admin interface using WebAuthn (Passkeys) and supports multiple isolated instances based on the request `Host` header.

# This should be a thing

## Project Overview
"This should be a thing" is a modern, multi-tenant Idea Board application. It allows users to propose ideas, vote, and discuss. It features a secure, passwordless admin interface using WebAuthn (Passkeys) and supports multiple isolated instances based on the request `Host` header.

## Tech Stack
*   **Framework:** Next.js 16 (App Router)
*   **Language:** TypeScript
*   **Database:** Neon Database (Serverless PostgreSQL) via `@neondatabase/serverless`
*   **UI:** Chakra UI v3, Lucide React
*   **Auth:**
    *   **Admin:** WebAuthn (Passkeys) via `@simplewebauthn`
    *   **User:** Email OTP via `resend` & JWT sessions

## Architecture & Data
*   **API Routes:** `src/app/api/` handles backend logic.
*   **Data Access:** `src/lib/` contains the data layer.
    *   `db.ts`: Database connection pool and schema initialization (`ensureSchema`).
    *   `store.ts`: CRUD operations for Ideas and Comments.
    *   `admin.ts`: Admin auth and data persistence.
    *   `auth.ts`: User authentication logic.
*   **Multi-tenancy:** All database tables (`ideas`, `comments`, `admin_data`, `auth_codes`) are partitioned by a `host` column.

## Guidelines

### 1. Database & Data Integrity
*   **Pattern:** Always use the dedicated helper functions in `src/lib/` (e.g., `store.ts`) instead of writing raw SQL in API routes.
*   **Security:** ALWAYS use parameterized queries (e.g., `VALUES ($1, $2)`) to prevent SQL injection.
*   **Multi-tenancy:** Every query MUST filter by `host` (e.g., `WHERE host = $1 ...`).
*   **Initialization:** Ensure `await ensureSchema()` is called before the first database operation in any flow.

### 2. Coding Standards
*   **Styling:** Use Chakra UI components. Maintain a clean, minimal design consistent with existing components.
*   **Type Safety:** Use strict TypeScript. Define interfaces for all data structures (e.g., `Idea`, `Comment`).
*   **Imports:** Use the `@/` alias for `src/` imports.

### 3. Workflow & Verification
*   **Build Check:** After significant changes, run `npm run build` to verify type safety and build success.
*   **Run unit tests:** Always ensure unit tests are passing before committing anything to git.

### 4. Git & Naming
*   **Commits:** Start messages with a lowercase verb (e.g., "fix bug in store", "add new component").
*   **Branding:** The project name is "This should be a thing" (only "This" is capitalized).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/abulujayn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
