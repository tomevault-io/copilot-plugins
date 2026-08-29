## poly-frontend-03b5e553

> React + Vite SPA. Read this before touching code.

# Agent instructions — `poly-frontend`

React + Vite SPA. Read this before touching code.

## Where to touch

- **Routes**: `src/app.tsx`.
- **Pages**: `src/pages/`.
- **Components**: `src/components/`. Specs colocated as `*.spec.tsx`.
- **Backend calls + types**: `src/lib/data-access-products.ts`. Base URL is `VITE_API_URL` (defaults to `http://localhost:3000`).
- **Styling**: Tailwind v4 via `@tailwindcss/vite`. Global entry `src/styles.css`.

## Tooling

- No Nx, no monorepo. Flat single-project layout at root.
- Tests: Vitest + jsdom, configured inline in `vite.config.ts`.
- Typecheck runs as part of `build` (`tsc -b && vite build`). No standalone typecheck script. To typecheck only: `npx tsc -b`.
- No ESLint, no Prettier. Don't reintroduce them unless asked.

## Conventions

- `verbatimModuleSyntax` is on. Use `import { type Foo }` for types.
- `react-router-dom` v6. Don't bump to v7 without reason.
- Don't add a Vite proxy; the frontend hits the backend at its absolute URL via `VITE_API_URL`.

## Verifying changes

```sh
npm run test       # vitest run
npm run build      # tsc -b + vite build
npm run dev        # http://localhost:5173 (needs backend on :3000 to load data)
```

---
> Source: [Poly-Demo/poly-frontend-03b5e553](https://github.com/Poly-Demo/poly-frontend-03b5e553) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
