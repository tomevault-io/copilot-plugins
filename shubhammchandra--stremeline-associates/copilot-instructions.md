## stremeline-associates

> AI automation agency website built with Next.js 16, React 19, TypeScript, Tailwind v4, in a pnpm Turbo monorepo.

# CLAUDE.md — Project Instructions for Claude Code

## Project: Stremeline Associates

AI automation agency website built with Next.js 16, React 19, TypeScript, Tailwind v4, in a pnpm Turbo monorepo.

## File Annotation Rule

All MAJOR files (pages, layouts, components, hooks, utilities, API routes, types, schemas, content) must have an annotation header after imports:

```typescript
/**
 * What this does: [One-line description]
 * Why it's here: [What problem it solves or what part of the app needs it]
 * How it works: [Brief explanation — 1-3 sentences]
 * Dependencies: [Key packages or internal modules]
 */
```

Skip annotations for: barrel exports (index.ts), config files, trivial one-liners.

Keep each field to 1-2 lines. Be concise. When creating or substantially editing a major file, always add or update the annotation.

## Monorepo Structure

### Apps

- `apps/web/` — Main Next.js 16 site (App Router, runs on port 3001)

### Packages — Application

- `packages/ui/` — Shared UI components built with Radix UI & CVA (@repo/ui)
- `packages/tokens/` — Design tokens (@repo/tokens)
- `packages/types/` — Shared TypeScript types (@repo/types)
- `packages/utils/` — Utilities: cn(), formatDate(), slugify(), createMetadata() (@repo/utils)
- `packages/validation/` — Zod form schemas for contact & newsletter (@repo/validation)
- `packages/content/` — Static content data: services, case studies, navigation (@repo/content)
- `packages/animation/` — Animation utilities & hooks for GSAP/Motion (@repo/animation)
- `packages/api-client/` — HTTP client layer (@repo/api-client)

### Packages — Configuration

- `packages/config-eslint/` — Shared ESLint configs (@repo/eslint-config)
- `packages/config-tailwind/` — Tailwind v4 CSS-first config (@repo/config-tailwind)
- `packages/config-typescript/` — Shared TypeScript configs (@repo/typescript-config)

## Key Conventions

- Server Components by default; `"use client"` only when needed.
- Tailwind v4 — config in CSS, not tailwind.config.js.
- Use `cn()` from `@repo/utils` for class merging.
- Strict TypeScript — no `any`.
- ESLint zero warnings policy.
- Import shared code via `@repo/*` workspace aliases.
- Pre-commit hooks via Husky + lint-staged (ESLint fix + Prettier).

## Deployment

- Hosted on **Vercel** — production URL: `stremeline-associates-sam-chands-projects.vercel.app`
- Vercel Root Directory is set to `apps/web`. The `vercel.json` lives in `apps/web/`.
- Build command: `cd ../.. && pnpm turbo build --filter=web` (runs from `apps/web`, navigates to repo root for Turbo).
- Production deploys trigger on push to `main`.
- Deployment protection is off for production (publicly viewable); SSO protection is preview-only.

## Commands

- `pnpm dev` — Dev server (web app on localhost:3001)
- `pnpm build` — Build all packages & apps
- `pnpm lint` — Lint all packages
- `pnpm check-types` — Typecheck all packages
- `pnpm test` — Run unit tests (Vitest)
- `pnpm test:e2e` — Run E2E tests (Playwright)
- `pnpm format` — Format with Prettier
- `pnpm format:check` — Check formatting without writing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ShubhamMChandra)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ShubhamMChandra)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
