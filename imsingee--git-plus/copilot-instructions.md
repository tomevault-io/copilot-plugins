## git-plus

> This file provides guidance to AI Code Agent when working with code in this repository.

# Repository Guidelines for AI

This file provides guidance to AI Code Agent when working with code in this repository.

## Project Structure & Module Organization

This repository is a Go backend plus a Vite/TanStack Router frontend workspace.

- Go entrypoints live at the repository root:
  - `main.go` is the thin CLI entrypoint
  - `frontend_dev.go` provides the development-mode frontend handler wrapper
  - `frontend_embed.go` provides the embedded production frontend handler wrapper for `-tags embed`
- Core Go packages live under `pkg/`:
  - `pkg/app/` for CLI setup and startup config
  - `pkg/server/` for server startup and HTTP route composition
  - `pkg/configservice/` for ConnectRPC handlers
  - `pkg/config/` for config loading and validation
  - `pkg/frontend/` for shared frontend handler implementations
- Protobuf and RPC sources live under:
  - `proto/` for `.proto` contracts managed by Buf
  - `pkg/rpc/` for generated Go protobuf and Connect stubs
- Frontend source lives in `frontend/`:
  - Vite entry: `frontend/src/main.tsx`
  - Router setup: `frontend/src/router.tsx`
  - Route modules: `frontend/src/routes`
  - Shared UI composites: `frontend/src/components`
  - Connect/Web generated client types: `frontend/src/rpc`
  - Shared Connect transport/helpers: `frontend/src/lib/connect`
  - Design primitives/themes: `frontend/src/ui`
  - Global styling: `frontend/src/styles.css`
  - Static assets: `frontend/public`
- Frontend production assets are emitted to `frontend/dist/`
- The production Go binary is emitted to `dist/git-plus`

Colocate new frontend feature assets with the component or route that consumes them to keep dependencies obvious.

- UI: Mantine v8. Use the context7 MCP tool with the library id `/mantine/mantine` to load docs.
- Routing: TanStack Router. Use context7 with `/websites/tanstack_router` for Router docs.
- Generated files like `routeTree.gen.ts` are auto-created by `@tanstack/router-plugin`; do not edit.
- Generated files under `pkg/rpc/` and `frontend/src/rpc/` are auto-created by Buf plugins; do not edit them by hand. Update `proto/` files and re-run code generation instead.

## Build & Development Commands

Use pnpm for workspace tasks and Go tooling for backend compilation/tests.

- `pnpm dev` starts both the frontend dev server and the Go server. The Go server proxies every non-`/api` request to the frontend dev server.
- `pnpm buf:generate` regenerates Go and TypeScript RPC code from `proto/` into `pkg/rpc/` and `frontend/src/rpc/`.
- `pnpm buf:lint` validates `.proto` files with Buf lint rules.
- `pnpm build` first builds the frontend into `frontend/dist/`, then builds `dist/git-plus` with `go build -tags embed`, embedding the frontend assets into the binary.
- `pnpm db:generate:drizzle` regenerates Drizzle SQL migrations from `db/src/schema.ts`.
- `pnpm db:generate:schema-sql` regenerates `db/schema.sql` from `db/src/schema.ts`.
- `pnpm db:generate:sqlc` regenerates Go `sqlc` query code from `db/schema.sql` and `db/queries/*.sql`.
- `pnpm db:generate` runs all database codegen steps in sequence.
- `pnpm test` runs `go test ./...` and then frontend Vitest.
- `pnpm check:types` runs `frontend` type-checking via `tsc --noEmit`.
- `pnpm lint` applies the TanStack + React ESLint rules.
- `pnpm format` runs Prettier, then ESLint autofix, then ESLint verification.

If you need frontend-only commands during local debugging, use the package-level scripts in `frontend/package.json` directly.

Database schema workflow:

- `db/src/schema.ts` is the source-of-truth for the entire SQLite schema.
- Do not handwrite or manually edit `db/migrations/*/migration.sql`.
- Do not handwrite or manually edit Drizzle snapshot files such as `db/migrations/*/snapshot.json`.
- Do not handwrite or manually edit `db/schema.sql`.
- When the database schema changes, update `db/src/schema.ts` first, then regenerate artifacts in order: `pnpm db:generate:drizzle`, `pnpm db:generate:schema-sql`, `pnpm db:generate:sqlc`.
- Always use `pnpm db:generate:schema-sql` to update `db/schema.sql`; do not edit it manually.
- Do not run `db/schema.sql` generation and `sqlc` generation in parallel. `sqlc` must read the freshly generated `db/schema.sql`.
- `db/schema.sql` is a generated downstream schema artifact for SQL tooling; do not treat it as the primary schema definition.
- Raw SQL execution is allowed only in the migration runner. All non-migration database reads and writes in application code must go through `sqlc`-generated queries defined in `db/queries/*.sql`.
- After changing database queries or any schema used by application code, regenerate database artifacts with `pnpm db:generate` unless you intentionally need only one sub-step.

Dependency upgrade policy:

- Keep `@types/node` and `eslint` at their current versions.
- Do not upgrade them unless explicitly requested.

Frontend testing uses Vitest:

- Vitest is configured via `vitest.config.ts` using `test.projects` (separate `unit` + `browser` projects).
- Root `pnpm test` runs Go tests first, then frontend Vitest.
- Frontend-only `pnpm --filter ./frontend test` runs all Vitest projects.
- Frontend-only `pnpm --filter ./frontend test:unit` runs the `unit` project.
- Frontend-only `pnpm --filter ./frontend test:browser` runs the `browser` project.
- The default test environment is `node`. For React component tests, prefer Browser Mode (`pnpm test:browser`) and name files `*.browser.test.tsx` / `*.browser.spec.tsx`.

Browser Mode notes:

- Test file pattern: `src/**/*.browser.{test,spec}.{ts,tsx}`
- Setup file: `vitest.browser.setup.ts`
- If Chromium is missing locally, install it with `pnpm exec playwright install chromium`

Before declaring any Agent task complete, re-run the commands affected by your changes. For full-stack changes, prefer validating with `pnpm test`, `pnpm build`, and `pnpm check:types`.
If you changed any `.proto` definitions or RPC wiring, also run `pnpm buf:generate` and `pnpm buf:lint`.

## ESLint

ESLint 9 flat config is used (`eslint.config.js`). The configuration extends `@tanstack/eslint-config` and adds React-specific rules via `eslint-plugin-react`.

Key React rules enforced (errors):

- `react/button-has-type` - buttons must have an explicit `type` attribute
- `react/self-closing-comp` - components without children must self-close
- `react/jsx-curly-brace-presence` - no unnecessary curly braces in JSX
- `react/jsx-boolean-value` - omit `={true}` for boolean props
- `react/jsx-fragments` - use `<>` syntax for fragments
- `react/no-children-prop` - don't pass children as a prop

Run `pnpm lint` to check for errors. ESLint errors are blocking and must be fixed before committing.

## Prettier

Prettier config lives in `prettier.config.js`:

- `semi: true` - always use semicolons
- `singleQuote: true` - use single quotes for strings
- `trailingComma: 'all'` - trailing commas everywhere

Run `pnpm format` to format and fix lint errors in one step (runs Prettier then ESLint --fix).

## Coding Style & Naming Conventions

Write TypeScript React function components. Prettier enforces semicolons, single quotes, and trailing commas—never hand-format around it.

Use PascalCase for components (`DashboardCard.tsx`), camelCase for hooks/utilities. Mantine styles should live beside their component, leveraging Mantine's theming utilities before reaching for raw CSS.

## Commit & Pull Request Guidelines

Commits should mirror the existing short, imperative pattern (`use mantine components for index page`). Keep changes scoped and meaningful. PRs must include: a concise summary, linked issue/ticket, before/after screenshots for UI work, and confirmation that `pnpm lint` and `pnpm check:types` passed. Mention any config or env changes up front so reviewers can reproduce.

## Security & Configuration Tips

Secrets belong at the repository root in local env files such as `.env` / `.env.local` (gitignored); frontend-only examples live alongside the workspace, but the active runtime setup is root-oriented. Run `lefthook install` once so git hooks catch lint regressions pre-push. Document any third-party scripts or analytics additions in the PR, including CSP or token requirements.

## Notes

1. Prefer Mantine components and styles; avoid unnecessary custom CSS.
2. Extract custom frontend components into `frontend/src/components/`; use the existing frontend structure and tooling rather than introducing a second pattern.
3. Always use English when writing code, comments, and documentation.

## Dependencies / Tools

### ConnectRPC + Buf

The backend API under `/api` is implemented with ConnectRPC and generated from protobuf definitions.

- Define API contracts in `proto/`.
- Generate Go server/client stubs into `pkg/rpc/` and TypeScript definitions into `frontend/src/rpc/`.
- Use Buf for schema linting and code generation via `pnpm buf:generate` and `pnpm buf:lint`.
- Mount Connect handlers under the `/api` base path. Frontend transports should use `createConnectTransport({ baseUrl: '/api' })`.
- Prefer changing schemas and regenerating code over editing generated RPC files directly.

### Mantine

The project uses [Mantine](https://mantine.dev/) v8 for UI components. Use the context7 MCP tool with the library id `/mantine/mantine` to load (or search) docs.

### Tanstack Router

The project uses [TanStack Router](https://tanstack.com/router/latest) with the Vite plugin for file-based routing. Use the context7 MCP tool with the library id `/websites/tanstack_router` to load (or search) docs.

### @mantine/hooks

Use [@mantine/hooks](https://mantine.dev/hooks/getting-started/) for common React hooks. Prefer these over writing custom hooks when available.

**Note:** Do NOT use `useClipboard` from @mantine/hooks. Use `copy-to-clipboard` instead (see below).

### copy-to-clipboard

Use [copy-to-clipboard](https://www.npmjs.com/package/copy-to-clipboard) for clipboard operations:

```tsx
import copy from 'copy-to-clipboard';

copy('Text to copy');
copy(text, { format: 'text/plain' });
```

### @mantine/modals

Use [@mantine/modals](https://mantine.dev/x/modals/) for declarative modal management. The ModalsProvider is already configured in `__root.tsx`.

### TanStack Query (React Query)

Use [TanStack Query](https://tanstack.com/query/latest) for server state management and data fetching. Use the context7 MCP tool with the library id `/tanstack/query` to load (or search) docs.

Prefer `useQuery` / `useMutation` / `useSuspenseQuery` / `useSuspenseInfiniteQuery` for data fetching. **Do NOT use `useEffect` + `useState` to fetch data.** TanStack Query provides built-in caching, background refetching, error handling, and loading states.

#### Integrating with Route Loaders

TanStack Query can be combined with TanStack Router's `loader` / `beforeLoad` for optimal data fetching:

1. **Define query options** using `queryOptions()` for reuse between loader and component
2. **Preload in loader** using `queryClient.ensureQueryData(options)` (awaits data) or `queryClient.prefetchQuery(options)` (non-blocking)
3. **Consume in component** using `useSuspenseQuery(options)` to automatically use cached data

Recommended patterns:

- Use `ensureQueryData` in loader for critical data that must be available before render
- Use `prefetchQuery` (without await) for slower/non-critical data, then wrap component in `<Suspense>` to handle loading
- Access `queryClient` from route context: `loader: ({ context: { queryClient } }) => ...`
- Always reuse the same `queryOptions` in both loader and component to ensure cache hits

### dayjs

Use [dayjs](https://day.js.org/) for date manipulation. It's a lightweight alternative to moment.js.

### Zod

Use [Zod](https://zod.dev/) v4 for schema validation. Prefer Zod for form validation and API response parsing.

### Sonner

The project uses [Sonner](https://sonner.emilkowal.ski/) for toast notifications instead of @mantine/notifications. Import `toast` from `sonner` to show notifications:

```tsx
import { toast } from 'sonner';

toast.success('Success message');
toast.error('Error message');
toast.info('Info message');
toast.warning('Warning message');
toast.loading('Loading...');
toast.promise(promise, {
  loading: 'Loading...',
  success: 'Done!',
  error: 'Error!',
});
```

### Context7

If you need to get some other documentations that you don't know exactly, you can use the context7 mcp tool.

---
> Source: [ImSingee/git-plus](https://github.com/ImSingee/git-plus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
