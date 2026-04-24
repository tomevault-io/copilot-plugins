## full-stack-template

> - Be direct, concise, and critical. Challenge my reasoning. No sycophancy.

# Style & Behavior

- Be direct, concise, and critical. Challenge my reasoning. No sycophancy.
- Don't include timeline estimates in plans.
- Don't add yourself as a co-author to git commits.

# Standards

- Always choose the correct fix over the quick fix. Technical debt compounds.
- Fix bugs when you find them if they affect current work. Don't defer, don't say "out of scope." Only exception: genuinely blocked by missing infrastructure.
- Never assume — read the code, verify, compare. Document findings with `file:line` references.
- When there's a tradeoff, present options with evidence and let me decide. Don't silently pick the easy path.
- Place documentation in `docs/` — context is lost between sessions.

# Project Architecture

Turborepo + pnpm monorepo. NestJS backend (CJS, port 3001) + Next.js frontend (ESM, port 3000).

- `apps/web/` — Next.js 16, React 19, Panda CSS + Park UI
- `apps/server/` — NestJS 11, Prisma (SQLite), Swagger
- `packages/ui/` — Park UI components + Panda CSS codegen + shared preset (`@repo/ui`)
- `packages/api-schema/` — OpenAPI spec, generated types, typed fetch client (`@repo/api-schema`)
- `packages/platform-logger/` — Shared structured logger
- `packages/config-*` — Shared ESLint, TypeScript, Vitest configs

Detailed docs: @docs/architecture.md, @docs/frontend.md, @docs/backend.md, @docs/panda-css.md, @docs/api-types.md

# Commands

```
pnpm dev                  # Start all apps (turbo)
pnpm build                # Build all packages
pnpm test                 # Run unit tests (vitest via turbo)
pnpm test:e2e             # Playwright e2e tests
pnpm check-types          # TypeScript type-check all packages
pnpm lint                 # ESLint all packages
pnpm generate:types       # Regenerate OpenAPI spec → TypeScript types
```

# Code Conventions

- Do not create barrel files (index.ts re-exports) — bad for tree shaking. Import from specific modules.
- `@repo/ui` component source files are compiled by Next.js via `transpilePackages`. Do NOT add a separate build step for components.
- Prefer Park UI components over raw `css()` calls. Use `Heading`, `Text`, `Button`, `Card.*`, `Field.*`, `Alert.*`, etc. from `@repo/ui/<component>`. Use layout components (`Container`, `Stack`, `Flex`) from `@repo/ui/jsx`. Reserve `css()` for one-off styles only.
- Import components per-module: `import { Button } from "@repo/ui/button"`, not from a barrel.
- NestJS DTOs: use explicit `@ApiProperty({ type: String })` — tsx doesn't support `emitDecoratorMetadata`.
- NestJS controllers: add `@ApiBody({ type: Dto })` for Swagger request body generation.
- Panda CSS: use `styled()` factory with `withProvider`/`withContext` from `createStyleContext`.
- Park UI `createStyleContext` is injected via `codegen:done` hook in `panda.config.ts`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/murphyjoseph) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
