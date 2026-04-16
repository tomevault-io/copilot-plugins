## all-access-artist

> Think carefully and only action the specific task I have given you with the most concise and elegant solution that changes as little code as possible. Do not refactor unrelated code. Minimize the blast radius of your changes and do not introduce new dependencies unless explicitly authorized.


# All Access Artist: Coding Agent Ruleset v1.0

## 1. The Principle of Minimal Intrusion (Mandatory)

Think carefully and only action the specific task I have given you with the most concise and elegant solution that changes as little code as possible. Do not refactor unrelated code. Minimize the blast radius of your changes and do not introduce new dependencies unless explicitly authorized.

## 2. Zero Trust Security and Validation

Security is **Priority 0**. Never trust client input.
* **Authentication:** All API endpoints (unless explicitly public) must validate the Supabase JWT via Hono middleware.
* **Validation:** Every input (body, params, query) must be validated and sanitized using `Zod` at the API boundary before any business logic is executed.
* **Authorization (RLS):** Always use a user-scoped Supabase client initialized with the request's Authorization header. Never use the Service Role key to bypass RLS for user operations.

## 3. Strict TypeScript Enforcement

The `any` type is forbidden. In the rare case it is required for third-party library compatibility, it must be tightly scoped and accompanied by a `// eslint-disable-next-line @typescript-eslint/no-explicit-any` comment explaining **why** it is necessary. All code must adhere to strict mode compliance. All variables, arguments, and return values must be explicitly typed. Utilize types generated from the Supabase schema for database interactions and types inferred from `Zod` schemas for API interactions.

## 4. State Management Principles

Strictly separate Server State from Client (UI) State.
* **Server State:** Use TanStack Query (React Query) exclusively for all asynchronous data fetching, caching, and mutations. Encapsulate these within custom hooks (e.g., `useReleases.ts`). The anti-pattern of fetching data in `useEffect` is forbidden.
* **Client State:** Use local component state (`useState`/`useReducer`) for state that is isolated to a single component and its direct children (e.g., an "is open" flag for a dropdown). Use Zustand for global client-side state that needs to be accessed by multiple, non-related components without prop-drilling (e.g., the currently logged-in user's profile, theme settings).

## 5. Component Architecture: SRP and Structure

Adhere to the Single Responsibility Principle (SRP). Components must do one thing well.
* **Presentational (Dumb) Components:** Focus purely on UI, utilizing `shadcn/ui` primitives. They receive data via props and emit events via callbacks. They must not contain data-fetching logic.
* **Container (Smart) Components/Pages:** Handle data fetching (using custom hooks), state management, and business logic orchestration.

## 6. Edge-Optimized Backend Architecture

Cloudflare Workers must be lean, fast, and organized.
* Keep dependencies minimal.
* Follow this structure within `backend/src/`: `routes/` for Hono route handlers, `services/` for reusable business logic, `types/` for `Zod` schemas and type definitions, and `middleware/` for Hono middleware.
* Route handlers (Hono) must be thin, focusing on validation and orchestration.
* Extract core business logic into the dedicated service layer (`src/services/`).
* Prefer pushing complex data filtering and processing down to PostgreSQL (via optimized queries or RPC) rather than processing data in Worker memory.

## 7. Data Integrity and Explicit Contracts

The boundaries between systems must be rigid and validated.
* **API Contracts:** Define `Zod` schemas for all API request/response bodies. Use these for backend validation and to infer TypeScript types on the frontend (End-to-End Type Safety).
* **Database Integrity:** Ensure data integrity at the database level using PostgreSQL constraints (Foreign Keys, Checks, Not Null).

## 8. Comprehensive Resilience and Error Handling

The application must handle failures gracefully.
* **Frontend:** Implement React Error Boundaries. Utilize TanStack Query's `isLoading`, `isError`, and `error` states to provide meaningful user feedback (Skeletons, Toasts, Error Messages). Use a toast library (e.g., `react-hot-toast`) to display user-facing error messages from the API.
* **Backend:** Return standardized, structured HTTP error responses. Responses should conform to a standard shape: `{ success: false, error: { message: string, code?: string } }`. Never leak stack traces or internal database details to the client.

## 9. Configuration and Secrets Management

No hardcoded secrets or magic strings in the codebase. All configuration (API URLs, keys, feature flags) must be injected via environment variables (e.g., `VITE_*` for the frontend, Worker bindings/secrets for the backend).

## 10. Code Consistency and Clarity

Prioritize readable, maintainable code over cleverness.
* Strictly adhere to the established design system (TailwindCSS, `shadcn/ui`). Do not introduce custom CSS or inline styles.
* Adhere to existing file structure, naming conventions, Prettier, and ESLint rules.
* Write self-documenting code. Comments should explain **why** a complex decision was made, not **what** the code is doing.
* All code will be automatically formatted on save by Prettier and linted by ESLint. The CI/CD pipeline will fail if linting or formatting rules are not met. This is enforced via pre-commit hooks (Husky & lint-staged).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brennanwesley) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
