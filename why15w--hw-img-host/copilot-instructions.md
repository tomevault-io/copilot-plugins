## hw-img-host

> - **pnpm only**: package manager pinned to `pnpm@11.0.9` in `package.json`. `pnpm install` to install.

# AGENTS.md

## Package manager & engine

- **pnpm only**: package manager pinned to `pnpm@11.0.9` in `package.json`. `pnpm install` to install.
- **Node**: `^20.19.0 || >=22.12.0`

## Commands

| Command           | What it does                                                                                                          |
| ----------------- | --------------------------------------------------------------------------------------------------------------------- |
| `pnpm dev`        | Dev server on `localhost:5173` (Vite HMR)                                                                             |
| `pnpm build`      | Runs `type-check` + `build-only` in parallel via `run-p`. Pass extra args to build via `pnpm build -- --mode staging` |
| `pnpm type-check` | `vue-tsc --build` (type-checks all tsconfig references)                                                               |
| `pnpm lint`       | `eslint . --fix` (flat config in `eslint.config.ts`)                                                                  |
| `pnpm format`     | `prettier --write src/` (src/ only)                                                                                   |
| `pnpm preview`    | Preview the production build locally                                                                                  |

## Build tooling quirks

- **Tailwind CSS v4** via `@tailwindcss/vite` Vite plugin (no PostCSS config or `tailwind.config.ts`). The import is in `src/assets/main.css` as `@import 'tailwindcss'`.
- **UI components**: shadcn-vue (`components.json`), style `new-york`, base `neutral`, with CSS variables in `src/assets/main.css`. Use `@/lib/utils` for the `cn()` helper. Add components via `npx shadcn-vue@latest add <name>`.

## Architecture

```
Browser (Vue 3 SPA)
  ├─ Password login → POST /api/auth/login
  │    └─ returns JWT token (7d expiry, stored in localStorage)
  ├─ Get upload signature → GET /api/upload/sign?name=&size= (auth required)
  │    └─ returns CNB PUT URL for client-side direct upload
  └─ Server-side upload → POST /api/upload/img (multipart/form-data, multer 20MB)
       └─ compresses server-side → uploads to CNB → returns proxy URLs

Image serving:
  GET /img-api/* (e.g. https://img.example.com/img-api/path/to/img.webp)
    └─ edge-functions/img-api/[[path]].ts (EdgeOne Edge Function)
         └─ proxies to CNB with CORS + 30s cache
```

- **Frontend**: Vue 3 + `<script setup lang="ts">` + Composition API. Three routes: `/` (HomeView), `/gallery` (GalleryView), `/login` (LoginView). Auth via `useAuth` composable (`src/composables/useAuth.ts`) — JWT stored in localStorage, axios interceptor adds Bearer token to all requests. Login redirect is handled by router guard in `src/router/index.ts`.
- **Backend API**: `node-functions/api/[[default]].ts` mounts two Express sub-routers:
  - `routes/auth.ts` — `POST /api/auth/login` (validates `UPLOAD_PASSWORD`, returns JWT)
  - `routes/upload.ts` — `GET /api/upload/sign` (auth required, returns CNB upload signature) + `POST /api/upload/img` (direct multer upload to CNB)
  - `_auth.ts` — JWT sign/verify using `UPLOAD_PASSWORD` as secret; `authMiddleware` for route protection
  - `_utils.ts` — `uploadToCnb()`, `signUpload()`, `buildImageUrl()`, `extractImagePath()`
  - `_reply.ts` — `reply()` helper, shape: `{ code, msg, data }` (code=0 is success)
- **Edge proxy**: `edge-functions/img-api/[[path]].ts` — catches `/img-api/*`, forwards to CNB with CORS headers.
- **`[[default]].ts` / `[[path]].ts`** naming is EdgeOne Pages convention (catch-all and dynamic route functions). Do not rename.
- **No tests exist** in this project.

## Environment variables (required for deployment)

These are set in EdgeOne console — not in code or `.env` files:

| Variable          | Example                                                                                |
| ----------------- | -------------------------------------------------------------------------------------- |
| `BASE_IMG_URL`    | `https://img.example.com/` (trailing slash required)                                   |
| `SLUG_IMG`        | `username/repo-name`                                                                   |
| `TOKEN_IMG`       | CNB personal access token                                                              |
| `UPLOAD_PASSWORD` | Upload password (doubles as JWT secret; if empty/unset, login and sign endpoints fail) |

## Code conventions

- **Prettier**: no semicolons, single quotes, 100 char print width, 2-space indent
- **LF** line endings (`.gitattributes` enforces `eol=lf`)
- **Path alias**: `@` → `./src` (configured in `vite.config.ts`, `tsconfig.json`)
- **VS Code**: use Volar (not Vetur), ESLint, Prettier extensions (see `.vscode/extensions.json`)

---
> Source: [WhY15w/hw-img-host](https://github.com/WhY15w/hw-img-host) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
