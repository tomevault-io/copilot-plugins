## ui

> - Brand as **elorm/ui** with lowercase `elorm` (mirroring shadcn/ui); public GitHub repo is **Akarikev/ui** (user repo, not an organization).

## Learned User Preferences

- Brand as **elorm/ui** with lowercase `elorm` (mirroring shadcn/ui); public GitHub repo is **Akarikev/ui** (user repo, not an organization).
- Prefer **Bun** for local dev and monorepo scripts.
- Prefer **native Next.js docs** over Mintlify; chose **Fumadocs core with a custom shadcn-style minimal layout**.
- Docs layout is minimal and shadcn-like, with understated headings and icons; prose must not double-border code inside `.not-prose` components (e.g. `InstallCommand`); when docs mention third-party libraries by name, link to their site in prose (`meta.docsLead`) and page headers (`meta.docsTitle` with markdown links).
- CLI should let users choose **Base UI, Radix UI, or HeroUI** at init; HeroUI keeps elorm's flat component API via wrappers, not HeroUI-native compound exports.
- Docs install snippets should cover **npm, pnpm, yarn, and bun**.
- Theme customization should be consolidated in one picker, not repeated across every demo section; marketing site defaults to **neutral** base and **mono** accent.
- Marketing site should use **Geist** fonts; header nav links should be **bold** (`font-semibold`); homepage stays minimal (hero + docs preview switcher + Base UI/Radix compare + footer).
- Component docs should include live preview and per-variant copy-paste code (shadcn-style); interactive demos start closed with an explicit open trigger.
- Elorm style rules: use `Field`/`FieldGroup` for forms, `gap-*` not `space-y-*`, icons in buttons use `data-icon`, semantic color tokens only, shared styles from `@/lib/ui-styles`; customize components with an **elorm soft identity** (rounded surfaces, soft shadows) rather than stock Radix/Base UI defaults; Select inside Dialogs needs `modal={false}` and a higher z-index on the positioner.
- Docs mobile navigation uses a **sheet menu** (hamburger); hide it on `**lg+`** desktop.
- Keep **README.md** project-focused with minimal docs; put open-source setup, workflows, and need-to-knows in **CONTRIBUTING.md**; do day-to-day work on **`dev`**, merge to **`main` via PR** when ready.

## Learned Workspace Facts

- Turborepo monorepo: `apps/www` (Next.js marketing + native `/docs`), `packages/cli`, `packages/schema`, `packages/registry`, `packages/registry-radix`, `packages/themes`.
- Docs content lives in `apps/www/content/docs/` and is served at `/docs` via Fumadocs (migrated from Mintlify proxy).
- `scripts/generate-docs.ts` generates component MDX from `registry.json` (supports `meta.docsLead` for body links and linked `meta.docsTitle` for page headers); marketing catalog counts come from `apps/www/lib/registry-catalog.ts`.
- Public site domain is **ui.elorm.xyz**.
- CLI config is `elorm.json` (`init`, `add`, `search`, `docs`, `build`); npm package is **`elorm`**, publish with `bun run publish:cli` (or `publish:cli:all` for npm + GitHub release via `scripts/create-github-release.ts`); CLI fetches components from **ui.elorm.xyz/r/** registry JSON, not git.
- Three uiLibrary registries: Base UI (`packages/registry`), Radix (`packages/registry-radix`), HeroUI wrappers (`packages/registry-heroui`).
- `apps/www` uses Next.js 16; run the site with `bun run --filter www dev`.
- Shared styling tokens live in `packages/registry/lib/ui-styles.ts` (mirrored in `apps/www/lib/ui-styles.ts`).
- Registry includes `icon-nav-link` (ui) + `page-footer` (block) for marketing footer patterns, `social-links` for configurable social icon link rows (github, x, mastodon, bluesky, reddit, discord), and opt-in `navii-avatar` mascot avatars via `@usenavii/react` ([Navii](https://navii.uxderrick.com/), `elorm add navii-avatar`); core `avatar` stays dependency-free.
- Agent skills live in `skills/`; install with `npx skills add Akarikev/ui --skill elorm` (skills.sh indexes from public GitHub install telemetry).
- Fumadocs MDX uses a static component map — don't nest custom tab children in MDX; use single registered components (e.g. `FrameworkSetup`).
- Docs search calls `/api/search?query=`; framework logos are local `NextMark`/`ViteLogo` in `framework-logos.tsx` (npm `geist` has no logos export).
- Hero and OG/Twitter image use `public/hero-anime-monitor.png`; hero uses a fixed-height container and `unoptimized` Next/Image to avoid stretch blur — do not add `app/opengraph-image.png` (it overrides layout metadata).

---
> Source: [Akarikev/ui](https://github.com/Akarikev/ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
