## 8-bitquest

> Astro 7 + Tailwind CSS v4 + TypeScript (strict) starter with typed, config-driven content and a

# AGENTS.md — Astro template

Astro 7 + Tailwind CSS v4 + TypeScript (strict) starter with typed, config-driven content and a
CSS-first token architecture. Single-language. Package manager: **pnpm**.

## Commands

| Command        | Action                                    |
| :------------- | :---------------------------------------- |
| `pnpm install` | Install dependencies                      |
| `pnpm dev`     | Dev server at `localhost:4321`            |
| `pnpm build`   | Production build to `dist/`               |
| `pnpm preview` | `wrangler dev` — the built Worker locally |
| `pnpm deploy`  | `astro build && wrangler deploy`          |
| `pnpm lint`    | ESLint                                    |
| `pnpm format`  | `eslint --fix` then Prettier              |
| `pnpm check`   | `astro check` (type `.astro`/`.ts`)       |
| `pnpm test`    | Every `*.test.ts` self-check under `src/` |

## Project structure

```
src/
├── components/
│   ├── Sections/<Page>/<Name>.astro  # layout-free page sections; Global/ for cross-page ones
│   ├── Cards/<Name>Card.astro        # composed, content-aware cards (on ui/pixel-panel)
│   ├── ui/<name>/<Name>.astro        # UI primitives (contract: ui/README.md)
│   └── svg/icons/                    # the icon system
├── config/
│   ├── siteData.json.ts            # typed site metadata (name, title, author, OG default)
│   ├── legalData.json.ts           # terms + privacy content
│   ├── siteSettings.json.ts        # siteLang/siteLocale + feature flags
│   └── types/configDataTypes.ts    # interfaces for the data files
├── data/<collection>/<slug>/       # content collection entries
├── js/                             # textUtils, schema (JSON-LD builders)
├── layouts/                        # BaseHead (SEO/meta), BaseLayout (shell)
├── pages/                          # thin route shells: own BaseLayout + SEO, compose Sections
└── styles/                         # global.css (entry), tailwind-theme.css, fonts.css
```

- **Sections vs Cards vs ui**: pages are thin route shells that own `BaseLayout` + SEO and compose
  **Sections** (layout-free content blocks, per-page or `Global/`); Sections build on **ui** primitives
  and **Cards** (content-aware compositions). Contracts: `src/components/Sections/README.md`,
  `src/components/Cards/README.md`, `src/components/ui/README.md`.
- `src/content.config.ts` — content collection schemas (Zod). Entries live directly under the
  collection dir (id `<slug>`).
- `src/config/` — typed site config. Content is three deliberate tiers: collections (`src/data/`,
  Zod-validated), config (`src/config/` — anything used on more than one page: brand, nav, legal,
  portfolio facts), and one-off section copy as a typed literal at the top of its Section component
  (the FAQ, tech list, gear table — edit the section to edit the copy). New cross-page values go in
  config, never as literals in components.
- Path aliases (`@config/* @js/* @layouts/* @components/* @assets/* @images/* @/*`) come from
  `tsconfig.json` `paths` — prefer them over deep relative imports.

## Stack defaults

- **TypeScript** strict; validate external data at the boundary (Zod).
- **Tailwind v4** CSS-first: tokens in `@theme`; use token utilities (`bg-primary`, `text-foreground`,
  `text-base-700`), never raw palette colors (`bg-violet-700`) — that bypasses theming + dark mode.
- **Astro 7** Rust compiler: close every tag, mind JSX whitespace (`{" "}`), default to zero-JS islands.

## Don't / gotchas

- **Set `SITE_URL` in the build environment** before a production deploy — `astro.config.mjs` falls
  back to the `https://example.com` placeholder, which feeds the sitemap and the canonical/OG URLs in
  `BaseHead.astro`. A production build (`DEPLOY_ENV=production`) throws on the placeholder.
- **Hosting is Cloudflare Workers** (`@astrojs/cloudflare` + `wrangler.jsonc`). The wrangler `main`
  must stay `@astrojs/cloudflare/entrypoints/server` — never a `dist/` path (it breaks `astro
check`). Server secrets (the Resend keys) go through the `astro:env` schema in `astro.config.mjs`,
  NOT `import.meta.env` — Workers runtime secrets never reach `import.meta.env`. Set them with
  `pnpm wrangler secret put <NAME>`; local dev reads `.env` as usual.
- **`vite.build.assetsInlineLimit: 0`** is intentional — inlined short scripts break under
  `<ClientRouter />` view transitions. Leave it at 0.
- **Theme is set pre-paint** by an inline script in `BaseHead` (follows the device
  `prefers-color-scheme`) — don't move it to a bundled `<script>` or you'll reintroduce a flash of the
  wrong theme.
- **Token discipline:** in markup use `bg-primary` / `text-foreground` / `text-base-700`, never raw
  `bg-violet-700` / `text-zinc-300` (bypasses theming + dark mode). See `tailwind-theme.css`.
- **Motion is owned, not vendored.** The `animate-*` catalog is `src/styles/motion/` — don't `pnpm add`
  an animation library. `prefers-reduced-motion` is handled by a global guard there; scroll-driven
  (`timeline-*`) elements still need `motion-reduce:animate-none`, and decorative motion is gated on
  `siteSettings.useAnimations`.
- **SEO is owned, not vendored.** `BaseHead` emits every meta/OG tag natively; structured data comes
  from the JSON-LD builders in `@js/schema`; `robots.txt`/`llms.txt` are dynamic endpoints. Don't
  `pnpm add` an SEO/robots/schema package.
- `.claude/memory.db` and `.claude/settings.local.json` are local state — gitignored, not artifacts.
- **Two deps are held a major behind — don't `pnpm update --latest` blindly.**
  - `typescript` stays on 6.x. TS 7 is blocked by two hard peer ranges: `@astrojs/check` (which is
    `pnpm check`) peers `typescript: ^5.0.0 || ^6.0.0`, and `typescript-eslint` (which is `pnpm lint`)
    peers `typescript: >=4.8.4 <6.1.0`. Upgrading breaks both commands until both release TS 7 support.
  - `eslint-plugin-astro` stays on 2.1.1. This one is **not** blocked — 3.0.1 declares the same peers
    and engines as 2.1.1 — it is simply untrialled, and a major on the plugin whose
    `configs.recommended` / `configs["jsx-a11y-recommended"]` this repo spreads into
    `eslint.config.mjs` can change that config surface. Upgrade deliberately and re-run `pnpm lint`,
    not as part of a sweep.
- **The content layer caches deleted entries.** `node_modules/.astro/data-store.json` survives
  `rm -rf .astro`, so removing a collection entry and rebuilding fails with
  `UnknownContentCollectionError` naming the file you just deleted. Clear both:
  `rm -rf node_modules/.astro .astro dist`. See `src/data/README.md`.
- **`/contact/` being on-demand duplicates the stylesheet.** The build emits `BaseLayout.<hash>.css`
  and a byte-identical `contact.<hash>.css`. It is an adapter artifact of mixing prerendered and
  on-demand routes (prerendering the route collapses them — verified), not something to fix in config.

## Verification

After non-trivial changes run the full chain:

```sh
pnpm lint && pnpm check && pnpm build && pnpm test
```

Schema and config mistakes surface at build time, so a clean build is the real check. `pnpm test`
discovers and runs every `*.test.ts` under `src/` with Node's type stripping (no framework, no
fixtures) and **fails when it finds none** — name a check `<thing>.test.ts` next to the code it covers
and it runs.

---
> Source: [Astro-Craft-Theme/8-BitQuest](https://github.com/Astro-Craft-Theme/8-BitQuest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
