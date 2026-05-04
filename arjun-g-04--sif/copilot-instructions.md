## sif

> This document provides essential information for AI agents operating in the SIF repository.

# Agent Guide: SIF (Sophisticated Instrumentation Facility)

This document provides essential information for AI agents operating in the SIF repository.

## 🚀 Commands

### Development & Build
- **Dev Server:** `pnpm dev` (runs on port 3000)
- **Build:** `pnpm build`
- **Preview:** `pnpm preview`
- **Type Check:** `pnpm exec tsc`

### Code Quality (Biome)
- **Lint & Format Check:** `pnpm check`
- **Fix Linting:** `pnpm lint`
- **Auto-format:** `pnpm format`
- **Linter Rule:** Biome is used instead of ESLint/Prettier.

### Testing (Vitest)
- **Run All Tests:** `pnpm test`
- **Run Single Test:** `pnpm vitest run src/path/to/file.test.ts`
- **Watch Mode:** `pnpm vitest`

### Development Workflow
- **Frequent Commits:** Agents are encouraged to make small, frequent commits while building features to track progress and allow for easy reverts.
- **Husky Hooks:** Pre-commit hooks may be temporarily disabled/commented out during intensive development to speed up the commit process. **IMPORTANT: They MUST be re-enabled before the final PR submission.**
- **Tidy Git:** After a PR is merged, agents should clean up the local environment when requested by the user: `git checkout main && git pull origin main && git fetch --prune`. Local feature branches that are already merged can be deleted using `git branch --merged main | grep -v '^\*' | xargs -n 1 git branch -d`.

### Database (Drizzle)
- **Generate Migrations:** `pnpm db:generate`
- **Apply Migrations:** `pnpm db:migrate`
- **Push Schema (Dev):** `pnpm db:push`
- **Studio (GUI):** `pnpm db:studio`

---

## 🛠 Architecture & Patterns

### Routing & Framework
- **Framework:** [TanStack Start](https://tanstack.com/router/v1/docs/guide/start/overview) (Full-stack React).
- **Routing:** File-based routing in `src/routes/`. 
- **Auto-generation:** `src/routeTree.gen.ts` is auto-generated. **DO NOT EDIT MANUALLY.**
- **Components:** Route components should be defined within the route file or imported from `src/components/`.

### Backend Logic (Server Functions)
- **Location:** `src/services/` or `src/lib/`.
- **Implementation:** Use `createServerFn` from `@tanstack/react-start`.
- **Validation:** Always validate inputs using `zod` and the `safeParseAndThrow` utility from `@/lib/utils`.
- **Auth:** Use `requireAdmin()` or `requireUser()` from `@/lib/auth` to protect server functions.

### Database (Drizzle ORM)
- **Schema:** Defined in `src/db/schema.ts`.
- **Access:** Import `db` from `@/db`.
- **Conventions:** Use `camelCase` for TypeScript field names and `snake_case` for database column names.
- **Transactions:** Use `db.transaction(async (tx) => { ... })` for multi-step operations.

---

## 🎨 Code Style & Conventions

### Formatting (Enforced by Biome)
- **Indentation:** Tabs (Width: 4).
- **Quotes:** Double quotes.
- **Semicolons:** Always required.
- **Import Sorting:** Managed by Biome.

### Naming Conventions
- **Components:** `PascalCase` (e.g., `BookingTable.tsx`).
- **Functions/Variables:** `camelCase`.
- **Files:** `camelCase` for utilities/services, `kebab-case` for config files.
- **Database Tables:** `camelCase` in exports, but pluralized (e.g., `export const users = ...`).

### Imports
- **Path Aliases:** Always use `@/` to refer to the `src/` directory.
- **Structure:**
  1. React/Framework imports
  2. External libraries (Zod, TanStack, etc.)
  3. Internal utilities/services (`@/lib`, `@/services`)
  4. Components (`@/components`)
  5. Styles/Assets

### Error Handling
- **Server Side:** Throw `new Error("message")` inside server functions; TanStack Start handles the serialization.
- **Validation:** Use `safeParseAndThrow(data, schema)` for consistent Zod error reporting.

### Components & UI
- **Styling:** Tailwind CSS v4.
- **Utility:** Use `cn()` from `@/lib/utils` for conditional class merging.
- **UI Components:** Found in `src/components/ui/` (shadcn/ui). Do not modify these directly; create wrapper components if customization is needed.
- **Icons:** Use `lucide-react`.

---

## 🔒 Security
- **Secrets:** Never commit `.env` files. Use `process.env` to access environment variables.
- **Auth:** Authentication is JWT-based, stored in HTTP-only cookies (`auth_token`).
- **CSRF/Protection:** Use Turnstile (Cloudflare) for public-facing forms (see `@/lib/turnstile`).

---

## 📝 Checklists for Agents

### Adding a New Route
1. Create file in `src/routes/path/to/route.tsx`.
2. Define `Route = createFileRoute('/path/to/route')({ ... })`.
3. Use `loader` for data fetching (server-side).
4. Run `pnpm dev` to regenerate `routeTree.gen.ts`.

### Adding a Server Function
1. Define in `src/services/` using `createServerFn`.
2. Define a Zod schema for input.
3. Validate with `safeParseAndThrow`.
4. Check permissions with `requireAdmin()` or `requireUser()`.

### Database Changes
1. Update `src/db/schema.ts`.
2. Run `pnpm db:generate`.
3. Run `pnpm db:migrate` or `pnpm db:push`.

---
> Source: [Arjun-G-04/sif](https://github.com/Arjun-G-04/sif) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
