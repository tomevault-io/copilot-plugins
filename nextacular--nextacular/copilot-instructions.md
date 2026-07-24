## nextacular

> Multi-tenant SaaS boilerplate for Next.js (Pages Router, TypeScript strict, Prisma 6, NextAuth 4, Stripe, Tailwind 3 + Headless UI 1.7). Optimize every change for downstream forkers — keep the surface area small and conventions consistent.

# Nextacular — Cursor rules

Multi-tenant SaaS boilerplate for Next.js (Pages Router, TypeScript strict, Prisma 6, NextAuth 4, Stripe, Tailwind 3 + Headless UI 1.7). Optimize every change for downstream forkers — keep the surface area small and conventions consistent.

Full guidance: see CLAUDE.md and docs/CONVENTIONS.md.

## Hard rules

1. Workspace-scoped API routes MUST call `requireWorkspaceOwner`, `requireWorkspaceMember`, or `requireMemberInOwnedWorkspace` from `src/lib/server/authorization.ts` before touching the database. Session presence alone is insufficient (this gap was a CVE in v1.4.2).
2. Files under `src/lib/server/` are server-only. Never import them from a client component or page that renders on the client.
3. `@prisma/client` is imported only from `prisma/services/*` and `prisma/index.ts`. Routes call services; services call Prisma.
4. API error responses use the shape `{ errors: { <field>: { msg: '<message>' } } }`. Client toasts depend on this.
5. Validate request bodies with `parseBody(schema, req.body, res)` from `src/lib/server/validate.ts`. Schemas live in `src/config/api-validation/` and use Zod.
6. TypeScript: `strict: true` and `noUncheckedIndexedAccess: true`. No `any`, no `@ts-ignore` without a one-line reason.
7. Use the `@/...` path aliases from `tsconfig.json`. No deep relative imports.
8. New environment variables: update `.env.sample` AND `docs/ENV.md`. Never bundle secrets behind `NEXT_PUBLIC_*`.

## Patterns to follow

- API routes: see `src/pages/api/workspace/[workspaceSlug]/name.ts` for the canonical `(validateSession + parseBody + requireWorkspaceOwner)` shape.
- Workspace-scoped pages: see `src/pages/account/[workspaceSlug]/settings/general.tsx` for the `getServerSideProps` pattern (session + `getWorkspace` + `isWorkspaceOwner` + workspace context).
- Data hooks: see `src/hooks/data/useMembers.ts` for the `useSWR` + typed-result shape.
- Compound components: see `src/components/Card/index.tsx` for the `FC` + named sub-components + `displayName` pattern.

## Don't

- Introduce a state-management library (the app uses React state + SWR + one Context).
- Replace the UI library mid-stream (Headless UI 1 + Tailwind is the contract until the planned shadcn migration in v2.0).
- Add raw SQL, Drizzle, or Kysely. Prisma services only.
- Use `defaultProps` on function components. Use destructured defaults.
- Disable `strict` or `noUncheckedIndexedAccess`.

---
> Source: [nextacular/nextacular](https://github.com/nextacular/nextacular) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
