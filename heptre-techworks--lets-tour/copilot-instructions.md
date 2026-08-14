## lets-tour

> Guidance for Claude Code (and other AI tools) working in this repository.

# CLAUDE.md

Guidance for Claude Code (and other AI tools) working in this repository.

## Start here

**Read [docs/KNOWLEDGE_MAP.md](docs/KNOWLEDGE_MAP.md) first.** It is the authoritative
orientation for this repo — tech stack, directory layout, the data model (Payload
collections), the block page-builder, plugins/hooks, caching, integrations, and a
"where do I change X" playbook. Also useful: [README.md](README.md) (setup) and
[admin-user-guide.md](admin-user-guide.md) (the CMS from a content-editor's view).

## What this is

Travel-agency website + headless CMS: **Next.js 15 (App Router, React 19)** +
**Payload CMS 3.53** + **MongoDB**, media on Vercel Blob, email via Zoho SMTP.
`src/payload.config.ts` is the central hub.

## Commands

```bash
pnpm dev              # dev server → http://localhost:3000  (admin at /admin)
pnpm build            # prod build (postbuild runs next-sitemap)
pnpm generate:types   # ★ run after ANY collection/global field change
pnpm lint             # ESLint
pnpm test             # Vitest (int) + Playwright (e2e)
```

> The README says `m s` to start the dev server — that is a typo. Use `pnpm dev`.

## Conventions & gotchas

- **Types are generated.** After editing any `src/collections/*` or `src/globals/*` field
  schema, run `pnpm generate:types` to refresh `src/payload-types.ts` (imported everywhere
  as `@/payload-types`). Don't hand-edit that file.
- **Import aliases:** `@/*` → `src/*`, `@payload-config` → `src/payload.config.ts`.
- **New block?** Create `src/blocks/<Name>/{config.ts,Component.tsx}`, register it in
  `src/blocks/RenderBlocks.tsx`, and add its config to the host collection/global.
- **Revalidation is automatic.** `autoRevalidatePlugin` injects cache-busting hooks into
  every collection/global; logic lives in `src/hooks/revalidateSite.ts` and pairs with the
  `unstable_cache` tags in `src/utilities/getGlobals.ts`.
- **Roles:** `customer` / `agent` / `admin` (see `src/collections/Users/index.ts`); lead
  collections rely on these for access control.
- **Secrets:** never commit real values. `.env.example` is placeholders only; keep it that
  way. Admin seeding in `payload.config.ts` `onInit` is env-driven — do not hard-code
  credentials.

## Git

- Work on a feature branch, not `main`.
- Only commit/push when the user asks.

---
> Source: [Heptre-Techworks/Lets-Tour](https://github.com/Heptre-Techworks/Lets-Tour) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
