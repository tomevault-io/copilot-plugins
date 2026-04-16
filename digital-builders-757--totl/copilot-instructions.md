## totl

> **BEFORE WRITING ANY CODE, YOU MUST:**

````markdown
# TOTL Agency – Cursor AI Assistant Rules

## 🎯 MANDATORY CONTEXT REFERENCE

**BEFORE WRITING ANY CODE, YOU MUST:**

1. **READ** `AGENT_ONBOARDING.md` (NEW AGENTS START HERE)
2. **READ** `TOTL_PROJECT_CONTEXT_PROMPT.md` completely
3. **READ** `docs/ARCHITECTURE_CONSTITUTION.md` (MANDATORY — system boundaries)
4. **CHECK** `supabase/migrations/` for current database schema
5. **VERIFY** RLS policies in the context file
6. **USE** generated types from `types/database.ts` / `@/types/supabase` (NO `any` types)
7. **FOLLOW** established project structure and naming conventions

If any of these files are missing from the context, proceed with best effort and flag the missing doc(s) in the plan (and any assumptions you’re making).

---

## 🧱 Architecture Constitution (MANDATORY)

Before implementing any feature, read:

- `docs/ARCHITECTURE_CONSTITUTION.md`

Non-negotiables:

- Middleware is security only; never create/update DB rows in middleware.
- Supabase Auth is identity; profiles is app identity; missing profile is a valid state during bootstrap.
- Mutations go through Server Actions / API routes only.
- Client components do not perform DB writes or direct privileged reads.
- RLS is the final authority; do not bypass with service role in client code.
- No `select('*')`; explicit columns.
- Never manually edit generated types.

If your change touches auth, middleware, redirects, profiles, onboarding, Stripe webhooks, or RLS:

1) Summarize the constitution in **5 bullets**.
2) Propose the change as a **minimal diff**.
3) List **risk points** and **tests** (redirect loops, bootstrap, RLS, webhook retries).

If modifying middleware/auth callback/profile bootstrap:

- Explain **why** the change is needed and **how it prevents redirect loops** (safe routes, signedOut handling, etc.).

If modifying Stripe webhooks:

- Must be **idempotent** (safe to retry), and must verify Stripe signatures.

## 🚨 RED ZONE FILES (NO BLIND REWRITES)

These files control **auth, routing, and schema truth**. They must **never** be auto-rewritten or heavily refactored in a single pass.

**Red Zone – Auth & Routing**

- `components/auth/auth-provider.tsx`
- `middleware.ts`
- `app/auth/callback/page.tsx`
- `app/talent/dashboard/page.tsx`
- `app/talent/dashboard/client.tsx`
- `app/client/dashboard/page.tsx`
- `app/client/dashboard/client.tsx`
- `app/(auth)/**` (login, signup, verification, choose-role, reset-password flows)
- `app/api/auth/*`
- `app/api/stripe/webhook/route.ts` (must be idempotent)
- `docs/SIGN_OUT_IMPROVEMENTS.md`
- `docs/contracts/AUTH_BOOTSTRAP_ONBOARDING_CONTRACT.md`
- `docs/TALENT_DASHBOARD_FLOW.md` (if present)

**Red Zone – Schema & Types**

- `database_schema_audit.md`
- `supabase/migrations/**`
- `types/database.ts`
- `types/supabase.ts` (or equivalent re-export)
- `docs/DATABASE_REPORT.md`

**Red Zone – Project Context**

- `TOTL_PROJECT_CONTEXT_PROMPT.md`
- `.cursorrules`

### Red Zone Editing Protocol (MANDATORY)

When modifying ANY Red Zone file:

1. **Show current file contents** (or at least the relevant section).
2. **Summarize the existing behavior** in plain language.
3. **Propose a small, explicit change** (minimal diff) – not a full rewrite.
4. **Apply only that change**, preserving existing working logic.
5. **Re-read** `docs/contracts/AUTH_BOOTSTRAP_ONBOARDING_CONTRACT.md`, `docs/SIGN_OUT_IMPROVEMENTS.md`, and `database_schema_audit.md` if the change touches:
   - auth state
   - middleware behavior
   - dashboards
   - database schema
6. **Never**:
   - Convert a Red Zone file into a “brand new pattern” all at once.
   - Remove working logic without explaining why.
   - Introduce `window.location.reload()` into auth/dashboards.

If a large refactor is truly needed, it must be broken into **small, reversible steps** with clear justification for each.

---

## 📋 PROJECT OVERVIEW

**TOTL Agency** – Talent booking platform connecting models/actors with casting directors and brands.

**Tech Stack:**

- Next.js 15.2.4 + App Router + TypeScript 5
- Supabase (PostgreSQL + Auth + Storage + Real-time)
- TailwindCSS + shadcn/ui components
- Resend API for custom emails

**Database:** PostgreSQL with RLS on all tables  
**Auth:** Supabase Auth with role-based access (talent/client/admin)  
**Key Tables:** `profiles`, `talent_profiles`, `client_profiles`, `gigs`, `applications`, `bookings`, `portfolio_items`

---

## 📋 PROJECT CONTEXT SUMMARY

- **Tech:** Next.js App Router + Supabase + TypeScript + shadcn/ui
- **Database:** PostgreSQL with strict RLS
- **Auth:** Role-based (`talent`, `client`, `admin`) tied to `profiles`
- **Security:** RLS enforced everywhere, no service keys in client
- **Architecture:** Server components for data fetching; presentational components only on the client

---

## ✅ COMPLIANCE CHECKLIST (BEFORE GENERATING CODE)

Before writing or editing any code, confirm:

- [ ] Read `TOTL_PROJECT_CONTEXT_PROMPT.md` for full context
- [ ] `database_schema_audit.md` is treated as a **schema audit** that must match `supabase/migrations/**` + generated `types/database.ts`
- [ ] Used proper TypeScript types (no `any`)
- [ ] Followed RLS-compatible query patterns
- [ ] Kept database logic **out of React components**
- [ ] Used centralized Supabase clients (`lib/supabase-client.ts` / `lib/supabase-admin-client.ts`)
- [ ] Implemented meaningful error handling
- [ ] Followed project naming and folder conventions
- [ ] For Red Zone files, followed the **Red Zone Editing Protocol**

---

## 🏗️ ARCHITECTURE RULES

### Database Access

- Use `lib/supabase-client.ts` for **client-side** queries (user-level access).
- Use `lib/supabase-admin-client.ts` for **server-side admin** operations only.
- Always assume **RLS is active** – do not attempt to bypass it.
- Use generated types from `@/types/supabase` / `types/database.ts`.

### Component Structure

- React components should be **presentational**.
- **No direct database calls** in components.
- Use **server components** and server actions for data fetching/mutations.
- Pass data into client components via **props** or typed hooks localized to the feature.

---

## 🔐 AUTHENTICATION & AUTHORIZATION

- **BEFORE ANY AUTH CHANGES:** Read `docs/AUTH_DATABASE_TRIGGER_CHECKLIST.md` and `docs/contracts/AUTH_BOOTSTRAP_ONBOARDING_CONTRACT.md`.
- Use `@supabase/auth-helpers-nextjs` patterns for session management.
- Check user roles before privileged actions.
- Use `middleware.ts` for route protection.
- Implement strong error handling for auth failures.
- Verify database triggers and constraints match `database_schema_audit.md`.

### Auth Ownership & Dashboards (CRITICAL)

- `AuthProvider` is the **single source of truth** for:
  - `session`
  - `user`
  - `profile`
  - `userRole`
  - `isLoading`
  - `isEmailVerified`
- Client components consume auth via `useAuth()` **only**.  
  No other place should fetch auth state directly from Supabase in the browser.

**Middleware responsibilities:**

- Gate by **session/role/suspension** only.
- Never create profiles or mutate data.
- Must allow onboarding-safe routes even when `profile` is `null`:
  - Auth pages (login/signup/verification)
  - Onboarding path
  - `/talent/dashboard` and `/client/dashboard` so `ensureProfileExists()` can run.

**Dashboards:**

- Dashboard **server pages** are thin wrappers:
  - They may fetch **page data** (gigs, bookings, etc.).
  - They do **not** own auth/profile logic.
- Dashboard **client components**:
  - Read `user`, `profile`, `userRole`, and `isLoading` from `useAuth()`.
  - Use a **single loading gate** combining auth + data-loading flags.
  - Avoid `window.location.reload()`.
  - Use `router.refresh()` only when strictly necessary (and never in a mount loop).

### Sign-Out Flow (Phase 5 Behavior)

Sign-out must follow this pattern:

1. **Reset auth state** in `AuthProvider`:
   - `user`, `session`, `userRole`, `profile`, `isEmailVerified`, `hasHandledInitialSession`, `isLoading`.
2. Optionally call `/api/auth/signout` (POST) to clear HTTP-only cookies server-side.
3. Call `supabase.auth.signOut()` from the browser client.
4. Call `resetSupabaseBrowserClient()` to avoid stale authenticated instances.
5. Redirect with:
   - `window.location.replace("/login?signedOut=true")` in the browser.
6. No aggressive manual cookie/localStorage purges or timers unless fixing a **specific documented bug**.

**Do not:**

- Change sign-out to use `window.location.reload()`.
- Introduce multiple competing redirect locations.
- Change `SIGNED_OUT` handling without explaining how it interacts with `signOut()`.

If updating `SIGNED_OUT` handling in `onAuthStateChange`:

- Treat **manual sign out** (user clicked “Sign Out”) as handled primarily by `signOut()`.
- Treat `SIGNED_OUT` as a safety net for:
  - Session expiry
  - Admin deletion
  - Cross-tab sync
- Avoid double redirects and loops.

---

## 🛡️ SECURITY BEST PRACTICES

- Never expose **service keys** in client code.
- Always validate that the current user has permission to access or modify data.
- Supabase queries are parameterized by default – do not use string concatenation to build queries.
- Respect the **least privilege** principle:
  - Use anon/user client whenever possible.
  - Use admin client only from trusted server code.

---

## 🚫 FORBIDDEN PATTERNS

- **Console statements in production code:**
  - ❌ `console.log()` / `console.debug()` - Use `logger.info()` / `logger.debug()` instead
  - ✅ `console.warn()` / `console.error()` - Allowed only for critical errors before logger init
  - ✅ Use `logger` utility (`lib/utils/logger.ts`) for all production logging
  - ESLint will error on `console.log/debug` - this is intentional

## 🚫 FORBIDDEN PATTERNS

- Using `any` types for database results or DTOs.
- Making database changes without updating `database_schema_audit.md`.
- Updating `types/database.ts` by hand (must be generated).
- Direct database calls in React components.
- Bypassing or weakening RLS policies.
- Exposing service keys to client/browser.
- Mixing database logic into UI components.
- Using raw SQL instead of the Supabase query builder (except in migrations/admin scripts where appropriate).
- Full-file rewrites of Red Zone files without going through the **Red Zone Editing Protocol**.

---

## 📁 CRITICAL FILES TO REFERENCE

- `TOTL_PROJECT_CONTEXT_PROMPT.md` – Complete project context
- `AGENT_ONBOARDING.md` – Quick start guide for agents
- `database_schema_audit.md` – Schema audit (must match migrations + generated types)
- `docs/AUTH_DATABASE_TRIGGER_CHECKLIST.md` – Mandatory for auth changes
- `docs/contracts/AUTH_BOOTSTRAP_ONBOARDING_CONTRACT.md` – Auth bootstrap + routing contract (canonical)
- `docs/contracts/INDEX.md` – All domain contracts (canonical)
- `docs/journeys/INDEX.md` – End-to-end journeys (canonical)
- `docs/OFF_SYNC_INVENTORY.md` – Winners declared + drift remediation tracker
- `docs/SIGN_OUT_IMPROVEMENTS.md` – Current sign-out behavior
- `types/database.ts` / `@/types/supabase` – Generated Supabase types
- `supabase/migrations/**` – Schema history
- `lib/supabase-client.ts` – Browser client config
- `lib/supabase-admin-client.ts` – Server admin client
- `middleware.ts` – Route protection logic
- `components/auth/auth-provider.tsx` – Authentication context owner

---

## 🔄 WHEN TO APPLY THESE RULES

These rules apply to:

- All code generation for this project
- Database schema changes or migrations
- New feature development
- Refactors and bug fixes
- API route creation or modification
- Component creation or updates
- Any changes touching auth, dashboards, or schema

---

## 🧬 DATABASE TYPES & CODEGEN

- `types/database.ts` is **auto-generated** from Supabase. Do not hand-edit.
- After any schema/migration changes, run:

  ```bash
  npx supabase gen types typescript \
    --project-id "$SUPABASE_PROJECT_ID" \
    --schema public > types/database.ts
````

* Prepend or keep an **AUTO-GENERATED** banner comment in `types/database.ts` to discourage manual edits.
* If there is a `types/supabase.ts` wrapper, it should only re-export from `types/database.ts`.

---

## 🧩 SUPABASE CLIENT USAGE

* Use a **single typed client** from `lib/supabase-client.ts` for user-level browser access.
* Use `lib/supabase-admin-client.ts` for server-side admin operations (service role).
* Do **not** import `@supabase/supabase-js` directly in pages/components.
* For server components/actions, use the project’s existing Supabase helpers (e.g., `createSupabaseServer` pattern if present).

---

## 🧮 QUERY STYLE

* Default: **explicit column selection**. Avoid `select('*')` in app code.
* Exception: internal admin scripts in `scripts/` may use `*` with care.
* Use `.maybeSingle()` when a row may be missing; use `.single()` only when you’re sure a row must exist and handle errors accordingly.
* For profile fetching, follow the established `fetchProfile` + `ensureAndHydrateProfile` pattern in `AuthProvider`.

---

## 🧱 SCHEMA MANAGEMENT

* **Single Source of Truth:** Always update `database_schema_audit.md` **before** changing schema.
* All schema modifications must:

  1. Be documented in `database_schema_audit.md`.
  2. Have a corresponding migration in `supabase/migrations`.
  3. Regenerate `types/database.ts`.
* Never modify the live database schema manually without a migration.
* Keep **audit file, live DB, and generated types** in sync.
* Use scripts/CI to check for drift where available.

---

## 🔒 TYPE SAFETY

* Use generated types from `@/types/supabase` / `types/database.ts` for all DB entities.
* Do not declare parallel interfaces/types for core tables (`profiles`, `gigs`, `applications`, etc.).
* No `any` for DB data:

  * If a result is typed as `any`, fix the query or import the right type.
* When changing an enum in the DB:

  * Update enum in `database_schema_audit.md`.
  * Regenerate types.
  * Update all string/union references in code.

---

## 🗄️ DATABASE ACCESS PATTERN

* All database calls must go through:

  * `lib/supabase-client.ts` for anon/user-level access
  * `lib/supabase-admin-client.ts` for admin/service-level access
* Do not instantiate new Supabase clients ad-hoc.
* RLS is always on for anon/user clients; rely on it.
* Admin client use must be explicit and server-only.

---

## ✅ VERIFICATION & CI

* Always run `scripts/verify-schema-sync.ps1` before commits/PRs.
* Fix:

  * Type mismatches
  * Outdated schema docs
  * `any` usages for DB data
  * Local interface duplication of DB entities
* CI will block PRs if:

  * Schema drift is detected
  * Types are out of sync
  * Forbidden patterns are introduced

---

## 💻 POWERSHELL ENVIRONMENT

* Use PowerShell commands:

  * `Get-ChildItem` instead of `ls`
  * `Get-Content` instead of `cat`
  * `Select-String` instead of `grep`

* All verification and automation scripts should be PowerShell-compatible.

---

## 📚 DOCUMENTATION-FIRST WORKFLOW

**Before starting ANY work:**

1. Check `docs/DOCUMENTATION_INDEX.md` to find relevant docs.
2. Read all relevant docs for the area you’re touching:

   * Auth → `docs/contracts/AUTH_BOOTSTRAP_ONBOARDING_CONTRACT.md`, `docs/AUTH_DATABASE_TRIGGER_CHECKLIST.md`, `docs/SIGN_OUT_IMPROVEMENTS.md`
   * Security → `docs/SECURITY_CONFIGURATION.md`
   * Database → `database_schema_audit.md`, `docs/DATABASE_REPORT.md`
   * Admin features → `docs/contracts/ADMIN_CONTRACT.md`
   * Feature implementation → related guides in `docs/`
   * Bug fixes → `docs/TROUBLESHOOTING_GUIDE.md`
3. Verify your approach aligns with documented patterns.

**After work:**

1. Update relevant documentation.
2. Add new docs for significant features.
3. Update `docs/DOCUMENTATION_INDEX.md` with new entries.

**All new docs go in `docs/`, not root.**

Root exceptions:

* `README.md`
* `database_schema_audit.md`
* `MVP_STATUS_NOTION.md`
* `notion_update.md`

### Documentation Date Format

**When creating new documentation files:**

1. **Always include a date field** at the top of the document:
   ```markdown
   **Date:** [Current Date]
   ```
   
2. **Date format:** Use full date format: `MMMM d, yyyy` (e.g., "December 15, 2025")
   
3. **How to get current date:** Run `Get-Date -Format "MMMM d, yyyy"` in PowerShell to get the current date

4. **Required fields for new docs:**
   - `**Date:** [Current Date]` - Always include
   - `**Status:**` - Current status (e.g., "✅ COMPLETE", "🚧 IN PROGRESS", "📋 PLANNED")
   - `**Purpose:**` or `**Scope:**` - What the doc covers

**Example header:**
```markdown
# Document Title

**Date:** December 15, 2025  
**Status:** ✅ COMPLETE  
**Purpose:** Description of what this document covers
```

---

## 🚨 CRITICAL ERROR PREVENTION – MANDATORY CHECKS

**Before pushing any code to `develop` or `main`:**

### 1. Schema & Types Verification

```bash
npm run schema:verify:comprehensive
npm run types:check
npm run build
```

### 2. Import Path Verification

**Never use:**

* `@/lib/supabase/supabase-admin-client` (WRONG – extra `/supabase/`)
* `@/types/database` (WRONG – should be `/types/supabase`)

**Always use:**

* `@/lib/supabase-admin-client`
* `@/types/supabase`

### 3. Common Errors to Avoid

* **Schema sync error:** `types/database.ts is out of sync with remote schema`
  → `npm run types:regen` for the correct environment.

* **Import path errors:** `Module not found: Can't resolve '@/lib/supabase/supabase-admin-client'`
  → Fix path to `@/lib/supabase-admin-client`.

* **Type errors:** `Property 'role' does not exist on type 'never'`
  → Ensure `Database` type is imported from `@/types/supabase`.

* **Build failures:** any non-passing local build
  → Do not push.

### 4. Branch-Specific Requirements

* `develop` branch: use `npm run types:regen:dev` if needed.
* `main` branch: use `npm run types:regen:prod` if needed.
* Both: must pass `npm run build` before pushing.

### 5. Emergency Fix Commands

```bash
# Schema sync error
npm run types:regen && npm run build

# Import path errors – find and fix manually
grep -r "@/lib/supabase/supabase-admin-client" . --exclude-dir=node_modules

# Build failures – fix locally first
npm run build
```

**Absolute rule:** Never push code that does not build locally.

---

## 📋 PRE-PUSH CHECKLIST

Always run this checklist before pushing:

1. ✅ `npm run schema:verify:comprehensive`
2. ✅ `npm run build`
3. ✅ `npm run lint`
4. ✅ Import paths are correct
5. ✅ Branch-specific types are generated
6. ✅ `docs/PRE_PUSH_CHECKLIST.md` reviewed and followed

If ANY step fails, fix it **before** pushing.

---

## 💻 CODE STYLE & STRUCTURE GUIDELINES

### TypeScript Code Style

* Use functional and declarative patterns; avoid classes.
* Prefer modular code over duplication.
* Use descriptive names with auxiliary verbs for booleans (`isLoading`, `hasError`, `canEdit`).
* File structure:

  1. Imports
  2. Types/interfaces
  3. Constants/helpers
  4. Main component/exports
  5. Subcomponents

**TypeScript usage:**

* Use `interface` for object shapes where possible.
* Avoid enums; use `const` objects + union types.
* Use `import type` when importing only types.
* Strict TypeScript mode is assumed and must remain clean.

```ts
interface ButtonProps {
  isLoading?: boolean;
  hasError?: boolean;
  onClick: () => void;
}

const Status = {
  ACTIVE: "active",
  INACTIVE: "inactive",
  PENDING: "pending",
} as const;

type Status = (typeof Status)[keyof typeof Status];
```

### Naming Conventions

* Files/directories: **kebab-case**

  * `talent-signup-form.tsx`
  * `apply-as-talent-button.tsx`
* Components: **PascalCase**
* Variables/functions: **camelCase**

---

### Syntax and Formatting

* Use `function` declarations for components and pure helpers.
* Use arrow functions for callbacks and small utilities.
* Prefer concise conditionals:

```tsx
if (isLoading) return <Spinner />;
if (hasError) return <ErrorMessage />;
```

---

### UI & Styling

* Use **shadcn/ui** and Radix primitives.
* Use TailwindCSS; no styled-components/CSS modules.
* Mobile-first responsive classes.

```tsx
<div className="flex flex-col gap-4 md:flex-row md:gap-8 lg:gap-12">
  <div className="w-full md:w-1/2 lg:w-1/3">...</div>
</div>
```

* For images, use Next.js `Image` or project `SafeImage` with explicit sizes.

---

### Performance & RSC

* Default to **Server Components**.
* Use Client Components only when needed:

  * Browser APIs
  * Event handlers
  * React hooks
  * Client-only libraries

```tsx
// Server Component
export default async function GigsPage() {
  const supabase = await createSupabaseServer();
  const { data } = await supabase.from("gigs").select("*");
  return <GigsList gigs={data} />;
}

// Client Component
"use client";
export function GigsList({ gigs }: { gigs: Gig[] }) {
  // ...
}
```

* Minimize `useEffect` and `useState`.
* Use Suspense and `loading.tsx` for loading states.
* Use `next/dynamic` for heavy components.

---

### URL Search Parameters

* In Server Components: use `searchParams` prop.
* In Client Components: use `useSearchParams()` safely with `useEffect`.
* Use optional chaining and nullish coalescing.

Refer to `docs/USESEARCHPARAMS_SSR_GUIDE.md` for patterns.

---

### Web Vitals

* Optimize LCP with prioritized hero images.
* Prevent CLS with explicit dimensions.
* Minimize JS, use code splitting, and lazy load non-critical UI.

---

### Code Quality & Error Handling

* Wrap async operations in `try/catch`.
* Use project error logging utilities (e.g., `lib/error-logger.ts`).
* Handle Supabase errors gracefully with meaningful messages.

---

### Accessibility

* Use semantic HTML.
* Add ARIA attributes when needed.
* Ensure keyboard navigation works.
* Rely on Radix for accessible primitives.

Before proposing or implementing a feature,
the agent must explain which Airport Zone(s) are affected
(Security, Ticketing, Manifest, Staff, Locks, Terminals, Control Tower)
and justify why.


---

**Final Principle:**
Prioritize **security**, **type safety**, and **auth stability** over stylistic preferences.
Red Zone files must be changed with precision, not rewritten on a whim.

```
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Digital-Builders-757) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
