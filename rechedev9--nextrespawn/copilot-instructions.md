## nextrespawn

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

---

# SaaS Starter — Agent Context

Next.js SaaS starter template.

## Stack (exact versions)

| Package | Version |
|---------|---------|
| Next.js | 16.2.2 |
| TypeScript | 6.0.2 |
| React | 19.2.4 |
| Tailwind CSS | 4.2 |
| shadcn/ui (`@base-ui/react`) | 4.1.2 |
| Auth.js / next-auth | 5.0.0-beta.30 |
| Prisma | 7.6.0 |
| Stripe | 22.0.0 |
| Resend + React Email | 6.10.0 |
| next-mdx-remote | 6.0.0 |
| Zod | 4.3.6 |

## Breaking changes vs. training data

### Next.js 16
- Middleware file is `proxy.ts`, not `middleware.ts`. Export: `proxy: NextProxy`. Config type: `ProxyConfig`.
- `instrumentation.ts` is stable — no config flag needed.

### Prisma 7
- No `url` or `directUrl` in `schema.prisma`. Connection URL lives in `prisma.config.ts`.
- `@prisma/adapter-pg` is required at runtime: `new PrismaClient({ adapter: new PrismaPg(pool) })`.

### shadcn/ui v4
- Uses `@base-ui/react`, not Radix. `asChild` does not exist — use the `render` prop.
- `buttonVariants` is in `lib/button-variants.ts` (no `"use client"`). Never import it from `components/ui/button`.

### Stripe 22
- Constructor requires `apiVersion: "2025-03-31.basil"` — omitting it is a TypeScript error.

### Auth.js v5
- Adapter: `@auth/prisma-adapter`, not `next-auth/adapters/prisma`.
- Session callback must explicitly copy `id` and `hasAccess` — they are not included by default.
- Session types are extended in `types/next-auth.d.ts` via module augmentation.

### Zod 4
- `.url()`, `.email()`, `.startsWith()`, `.endsWith()` are deprecated — use `.regex()`.
- `.flatten()` is removed — use `.issues.map(i => ...)`.
- Always `safeParse()` in API routes.

### next-mdx-remote 6
- Import from `next-mdx-remote/rsc` for Server Components, not from root.

## Key file locations

```
config.ts               — single source of truth: appName, pricing, FAQ, social
lib/auth.ts             — NextAuth config (Google + Resend + PrismaAdapter)
lib/prisma.ts           — DB singleton (Pool + PrismaPg adapter)
lib/stripe.ts           — Stripe lazy singleton
lib/env.ts              — Zod env validation (called by instrumentation.ts on startup)
lib/button-variants.ts  — buttonVariants CVA — no "use client", safe in RSC
prisma.config.ts        — Prisma 7 connection config (URL here, not schema.prisma)
proxy.ts                — Next.js 16 middleware (rate limiting only)
types/next-auth.d.ts    — Session/User type augmentation
```

## Critical patterns

**Route protection** is in `app/(dashboard)/layout.tsx`, not in `proxy.ts`:
```ts
const session = await auth();
if (!session) redirect("/login");
```

**Lazy singletons** — Stripe and Resend must not initialize at module load time (SSG fails). Use the `let _client = null` getter pattern in `lib/stripe.ts` and `lib/resend.ts`.

**HMAC comparison** — always `timingSafeEqual` from `node:crypto`, never `===` for digest comparison.

**Checkout price validation** — `/api/checkout` whitelists `priceId` against `config.pricing`. Never trust client-submitted price IDs.

## TypeScript rules

- No `any` — use `unknown` with type guards
- No non-null assertions (`!`)
- No `@ts-ignore` / `@ts-expect-error`
- All exported functions must declare return types
- `readonly` in interfaces unless mutation is required
- Run `npm run build` to verify — compiler is truth, not the LSP

## Commit rule

Use `./committer "type(scope): message" file1 file2` — never bare `git commit`.

---
> Source: [rechedev9/nextrespawn](https://github.com/rechedev9/nextrespawn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
