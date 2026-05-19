## react-typescript

> React/TypeScript and Inertia conventions for resources/js.


# React / TypeScript

- Prefer delegating to the `frontend` subagent for substantial frontend work.
- Use functional components and TypeScript interfaces for props.
- Imports: `@/components`, `@/lib/utils`, `@/hooks`, `@/types` (see tsconfig/vite aliases).
- Inertia: `Head`, `router`, `usePage` from `@inertiajs/react`; type page props from controller payload.
- Prettier: single quotes, tabWidth 4 — see .prettierrc.
- Reference: resources/js/pages/dashboard.tsx, resources/js/types/index.ts.

---
> Source: [andrejvysny/spendly](https://github.com/andrejvysny/spendly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
