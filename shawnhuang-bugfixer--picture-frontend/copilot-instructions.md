## picture-frontend

> Conventions and repo-specific notes for agentic coding assistants.

# AGENTS.md

Conventions and repo-specific notes for agentic coding assistants.

## Project Snapshot

- Stack: Vue 3 + TypeScript, Vite, Ant Design Vue, Pinia, Vue Router
- HTTP: Axios instance in `src/request.ts` (used by generated OpenAPI clients)
- Styling: Less (see `src/styles/variables.less`)

## Important Entry Points

- App bootstrap: `src/main.ts` (registers plugins + a `v-lazy` directive)
- Router: `src/router/index.ts`
- Global access guard: `src/access.ts`
- Axios instance + auth refresh: `src/request.ts`

## File Layout (High Level)

```
src/
  api/          # OpenAPI generated clients + types
  components/   # Reusable components
  constants/    # Enums/maps
  layouts/      # Layout shells
  pages/        # Route pages
  router/       # Vue Router config
  stores/       # Pinia stores
  styles/       # Less variables
  utils/        # Utilities (websocket/sse/etc)
```

## Commands

### Install / Dev

- `npm install`
- `npm run dev` (Vite dev server; default http://localhost:5173)
- `npm run preview` (serve production build locally)

### Build / Type Check

- `npm run build` (runs `npm run type-check` + `npm run build-only` in parallel)
- `npm run type-check` (vue-tsc project build)
- `npm run build-only` or `npm run pure-build` (Vite build without type-check)

### Lint / Format

- `npm run lint` (ESLint with auto-fix)
- `npm run format` (Prettier write; scoped to `src/`)

Targeted runs (useful for agents):

- `npx eslint "src/pages/HomePage.vue" --fix`
- `npx prettier --write "src/pages/HomePage.vue"`

### API (OpenAPI)

- `npm run openapi` (regenerates `src/api/*`; requires backend at `http://localhost:8123/api/v2/api-docs`)

### Tests

- No automated test runner is configured (no `npm test`). Validate changes manually.
- Single-test equivalents (only if a test runner is added later):
  - Vitest: `npx vitest "path/to/test" -t "test name"`

## Repo Rules Files

- No Cursor/Copilot rule files found: `.cursor/rules/`, `.cursorrules`, `.github/copilot-instructions.md`.

## Formatting And File Hygiene

- `.editorconfig`: UTF-8, 2-space indent, trim trailing whitespace, final newline.
- Prettier (`.prettierrc.json`): no semicolons, single quotes, `printWidth: 100`.
- Prefer changing only relevant lines; do not reformat unrelated files.
- If you touch formatting-heavy files, run `npm run format` afterward.

## Imports

- Prefer `@` alias imports for anything under `src/` (configured in `vite.config.ts` and `tsconfig.app.json`).
- Import grouping: external deps first, then `@/` local imports.
- Repo convention is to include explicit `.ts` extensions on local TS imports
  (example: `import { useLoginUserStore } from '@/stores/useLoginUserStore.ts'`).
  Exception: generated code and tooling config may omit extensions.
- Use `import type { ... }` for type-only imports when it helps avoid runtime cycles.

## TypeScript Conventions

- API types are under the global `API` namespace from `src/api/typings.d.ts`.
- Prefer concrete types over `any`; use `any` only at integration boundaries.
- Keep reactive state typed:

```ts
const formState = reactive<API.UserLoginRequest>({
  userAccount: '',
  userPassword: '',
})
```

## API Calls And Error Handling

- API clients are generated in `src/api/*Controller.ts`; do not edit these manually.
- Success convention:

```ts
const res = await userLoginUsingPost(values)
if (res.data.code === 0 && res.data.data) {
  message.success('操作成功')
} else {
  message.error('操作失败，' + res.data.message)
}
```

- Use `message` from `ant-design-vue` for user-facing feedback; include backend `res.data.message` when present.

## HTTP Layer (Axios)

- Use the shared Axios instance from `src/request.ts` (cookie auth enabled via `withCredentials: true`).
- Auth refresh behavior: when response `data.code === 40100`, the interceptor attempts
  `POST /api/user/auth/refresh`, and redirects to `/user/login` if refresh fails.
- Avoid creating ad-hoc Axios instances unless there is a strong reason.

## Routing And Access Control

- Router is defined in `src/router/index.ts`.
- Global route guard lives in `src/access.ts`:
  - First navigation fetches current user (`useLoginUserStore().fetchLoginUser()`)
  - `/admin/*` requires `loginUser.userRole === 'admin'` and otherwise redirects to login
- Route names may be Chinese; preserve existing naming.

## State Management (Pinia)

- Stores are in `src/stores/` and use the setup-style `defineStore`.
- Pattern: keep state in `ref`, async fetch methods named `fetchXxx`, and a setter when needed.

## Realtime (WebSocket / SSE)

- See `WEBSOCKET_README.md` for usage.
- The app uses `useWebSocketStore` (`src/stores/useWebSocketStore.ts`) with a WebSocket-first strategy
  and SSE fallback (`src/utils/sse.ts`).
- Login flow / initial bootstrapping is orchestrated via `src/access.ts`.

## Vue SFC Patterns

- Prefer `<script setup lang="ts">`.
- Keep sections in this order: imports, props, emits, composables, state, computed/watch, methods, lifecycle.
- Prefer `<style scoped lang="less">` and reuse variables from `src/styles/variables.less`.
- Use `IntersectionObserver` where already established for infinite scroll / lazy loading.

## Naming

- Components/pages: PascalCase filenames (e.g. `src/pages/HomePage.vue`).
- Stores: `useXxxStore` in `src/stores/`.
- Variables/functions: camelCase; constants/enums: UPPER_SNAKE_CASE.
- CSS classes: kebab-case.

## Generated / Do-Not-Edit

- `src/api/**` and `src/api/typings.d.ts` are generated; change the backend spec and rerun `npm run openapi`.

---
> Source: [ShawnHuang-Bugfixer/picture-frontend](https://github.com/ShawnHuang-Bugfixer/picture-frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
