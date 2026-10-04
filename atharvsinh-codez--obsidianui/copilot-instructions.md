## obsidianui

> Read `ACEBUILDER.md` for the user's saved shadcn, Aceternity, theme, accessibility, and motion conventions. Use the nine repository-local skills listed there from `skills/`, applying each within its stated scope.

# Project instructions

Read `ACEBUILDER.md` for the user's saved shadcn, Aceternity, theme, accessibility, and motion conventions. Use the nine repository-local skills listed there from `skills/`, applying each within its stated scope.

This checkout is ObsidianUI, a component library and documentation site. The user authorized the upgrade to Next.js 16.3.4, React 19, Tailwind CSS v4, and Nextra 4 with App Router. The Ara paths in the supplied reference remain template examples; this product is ObsidianUI.

Keep standard shadcn primitives in `src/components/ui`, reusable custom components in `src/components/block`, landing sections in `src/components/landing`, shared site chrome in `src/components/site`, media helpers in `src/components/media`, and route feature roots in `src/components/pages`. Docs content lives in `src/content`; route files in `src/app` stay thin. Tooling belongs in root `scripts` and regression tests in `tests`.

Read existing code before editing, preserve the current brand, keep changes targeted, and verify with available checks. Never claim unavailable platform tools ran. Do not run `npx shadcn add`; the user's installation convention is the platform registry tool. Never write `.env*` files or expose secrets.

`PROJECT_REVIEW.md` records the initial orientation and subsequent fixes. Recheck observations against current code. Run `npm run check` for lint, type checking, tests and production build. Regenerate `public/r` from source using `npm run registry:build`; never hand-edit generated manifests.

<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->

---
> Source: [Atharvsinh-codez/ObsidianUI](https://github.com/Atharvsinh-codez/ObsidianUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-04 -->
