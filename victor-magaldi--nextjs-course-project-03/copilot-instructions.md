## project-stack-versions

> Project stack, versions, and best practices

# Project Stack and Versions

This rule documents the current project stack to keep technical consistency and avoid changes that conflict with the versions in use.

## Runtime and scripts

- Node.js project using `npm` and scripts:
  - `npm run dev` -> `next dev`
  - `npm run build` -> `next build`
  - `npm run start` -> `next start`
  - `npm run lint` -> `next lint`

## Application dependencies

- `next@14.0.2`
- `react@^18`
- `react-dom@^18`
- `next-auth@5.0.0-beta.3`
- `@auth/core@0.18.1`
- `@auth/prisma-adapter@1.0.6`
- `@prisma/client@^5.21.1`
- `prisma@^5.21.1`
- `@nextui-org/react@^2.4.8`
- `framer-motion@^11.11.10`
- `zod@^3.23.8`

## Development dependencies

- `typescript@^5`
- `@types/node@^20`
- `@types/react@^18`
- `@types/react-dom@^18`
- `eslint@^8`
- `eslint-config-next@14.0.2`
- `tailwindcss@^3.3.0`
- `postcss@^8`
- `autoprefixer@^10.0.1`

## Best-practice notes by stack

### Next.js 14 + React 18

1. Prefer Server Components by default and use Client Components only when local state, effects, or browser APIs are required.
2. Use `loading.tsx`, `error.tsx`, and App Router segment boundaries for a more resilient UX.
3. Avoid duplicated fetches: centralize data access on the server and use cache/revalidate intentionally.

### NextAuth v5 beta

1. Treat it as an evolving API: before upgrades, review beta-version breaking changes.
2. Centralize auth configuration (providers, callbacks, and adapter) to reduce drift across routes.
3. Never expose secrets on the client; keep sensitive variables only on the server and in environment configs.

### Prisma 5

1. Ensure `prisma` and `@prisma/client` move together to avoid runtime incompatibilities.
2. In development, reuse a global Prisma Client instance to prevent excessive connections.
3. Recommended flow: use `prisma migrate dev` for schema evolution and validation before merge.

### TypeScript 5 + Zod

1. Validate external input with Zod (request body, params, envs) and infer types from schemas.
2. Avoid `any`; prioritize domain-derived types and explicit return types for service functions.
3. For server/client contracts, keep schemas and types in the same module to reduce drift.

### Tailwind CSS 3 + NextUI 2 + Framer Motion 11

1. Maintain visual consistency: define per-component styling responsibility (Tailwind utilities vs NextUI props).
2. Avoid excessive and repeated utility classes; extract patterns into reusable components.
3. For Framer Motion animations, prioritize short, accessible transitions and respect reduced-motion preferences.

## Upgrade policy

1. In dependency PRs, update the lockfile and record expected impact (build, lint, auth, db, and UI).
2. For beta dependencies (e.g., `next-auth@5.0.0-beta.3`), require manual validation of critical flows.
3. Avoid large batched upgrades without tests; prefer smaller domain-based batches (auth, UI, infra, lint).

---
> Source: [victor-magaldi/nextjs-course-project-03](https://github.com/victor-magaldi/nextjs-course-project-03) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-30 -->
