## client-management-system

> These rules guide implementation quality, performance, and organization.


# Engineering Rules & Conventions

These rules guide implementation quality, performance, and organization.

## Core Rules (from Product Owner)
- Best code possible with real execution and implementation; aim for enhanced, efficient, and very fast loading using modern tech.
- Code like a senior React/Next.js engineer: concise but correct; avoid cleverness that hurts clarity.
- Organize components by page and by use; create subfolders with `index.ts` barrel exports.
- Ensure the whole page loads when fetching data (SSR/RSC-first); avoid empty shells.
- Create APIs based on the pages, and place business logic in `src/lib/actions/*`.
- Always implement `index.ts` in component and actions folders for simple imports.

## Directory & Naming
- `src/lib/actions/<domain>/<action>.ts` with `index.ts` exporting all domain actions.
- `src/components/<page-or-shared>/*` with `index.ts`.
- `src/app/api/<domain>/...` route handlers mirror actions.
- Filenames: `camelCase` for actions and utilities, `PascalCase` for React components.

## Type Safety & Validation
- TypeScript `strict: true`.
- Validate all external inputs with Zod in actions and API.
- Never trust client requests; re-check permissions on server.

## Performance
- Prefer RSC and SSR; minimal client JS.
- Use pagination, indexes, and `select` to reduce payloads.
- Use caching with `revalidate` for safe reads; invalidate after mutations.
- Use `loading.tsx` and Suspense for secondary content only.

## API & Actions
- Single source of truth in actions; API handlers import actions.
- Consistent error shape `{ error: { code, message, details? } }`.
- Return typed results with discriminated unions for success/error in actions.

## UI/UX
- Use shadcn/ui and Tailwind; follow accessibility best practices.
- Componentize forms and lists; keep components small and testable.

## Testing & Quality
- ESLint + Prettier enforced; CI must pass.
- Vitest is the testing framework.
- Every new API route under `src/app/api/...` MUST include a corresponding test under `src/tests/api/...` mirroring the route path.
- Every page backed by server actions MUST ensure those backends (actions/API) have tests.
- Mock `@/lib/actions` for API route tests; mock `@/lib/db` and external SDKs for action tests.
- Keep error shape consistent in tests: `{ error: { code, message, details? } }`.
- e2e smoke tests for critical flows are optional initially.
- Protect main branch; PRs require review.

## Security
- Auth required for creator routes; per-action authorization by role.
- Share links: revocation, optional password, and rate limiting.

## Example Barrel Export
```ts
// src/lib/actions/projects/index.ts
export * from "./createProject";
export * from "./getProjects";
export * from "./updateProject";
export * from "./getProjectById";
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zafajardo9) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
