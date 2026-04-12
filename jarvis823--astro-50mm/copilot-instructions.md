## astro-50mm

> - **Framework:** Astro 5.x (static by default, per-page SSR opt-in)

# Astro 50mm — Project Conventions

## Stack
- **Framework:** Astro 5.x (static by default, per-page SSR opt-in)
- **Deploy target:** Cloudflare Workers
- **Styling:** CSS variables + Astro scoped styles (no Tailwind)
- **Content:** Astro Content Collections (Markdown/MDX)
- **TypeScript:** Strict mode

## Commands
```bash
npm run dev        # Start local dev server
npm run build      # Production build
npm run preview    # Build + wrangler dev (local preview)
npm run deploy     # Deploy to Cloudflare Workers
npm run check      # Build + tsc + deploy dry-run
npm run cf-typegen # Regenerate Cloudflare types after changing wrangler.json bindings
npm run lint       # ESLint check
npm run format     # Prettier format
```

## Project Structure
```
src/
├── assets/              # Images/icons processed by Astro pipeline
├── components/
│   ├── common/          # Shared structural (Header, Footer, Nav, BaseHead)
│   ├── ui/              # Reusable primitives (Button, Card, Container)
│   ├── landing/         # Landing page sections (Hero, Features, Pricing)
│   ├── blog/            # Blog-specific components
│   └── mdx/             # Custom MDX components
├── content/
│   ├── blog/            # Blog posts (md/mdx)
│   └── pages/           # CMS-driven pages
├── layouts/
│   ├── Base.astro       # html/head/body shell
│   ├── Page.astro       # Base + Header/Footer
│   ├── BlogPost.astro   # Page + article chrome
│   └── Landing.astro    # Page + full-width sections
├── lib/                 # Shared utilities (cloudflare.ts, utils.ts)
├── pages/               # File-based routing
│   └── api/             # SSR API endpoints
├── styles/
│   ├── tokens.css       # Design tokens (CSS variables)
│   ├── global.css       # Base styles + imports tokens/utilities
│   └── utilities.css    # .sr-only, .container, .prose
├── types/               # Shared TypeScript types
├── consts.ts            # Site-wide constants
├── content.config.ts    # Content collection schemas
└── env.d.ts             # Cloudflare runtime types
```

## Naming Conventions
- **Components:** PascalCase `.astro` — `PostCard.astro`, `Hero.astro`
- **Pages/routes:** kebab-case — `about.astro`, `blog/[...slug].astro`
- **API routes:** `.ts` — `api/contact.ts`
- **TS library files:** camelCase — `cloudflare.ts`, `utils.ts`
- **CSS variables:** `--category-name` — `--color-primary`, `--space-md`
- **Content files:** kebab-case — `first-post.md`
- **Images:** kebab-case — `hero-background.jpg`
- **Collections:** singular — `blog`, `page`

## Import Convention
Use `@/` path alias for all imports:
```ts
import Header from '@/components/common/Header.astro';
import { SITE_TITLE } from '@/consts';
import type { NavItem } from '@/types';
```

## Layout Chain
Every page uses one layout: `Base → Page → BlogPost/Landing`
- `Base.astro` — html/head/body only
- `Page.astro` — Base + Header + main + Footer
- `BlogPost.astro` — Page + article wrapper
- `Landing.astro` — Page without main wrapper (full-width sections)

## Component Rules
- **Default to `.astro`** — zero client JS
- Always define `interface Props` in frontmatter
- Accept optional `class` prop on wrapper components
- Use `class:list` for conditional classes
- Use named slots for layout regions

## JavaScript Restrictions (Cloudflare Workers)
**CRITICAL — Workers have strict limits:**
- Bundle size: 10MB compressed
- CPU time: 10ms (free) / 30s (paid) per request
- No `eval()` or `new Function()`
- No dynamic `import()` in SSR code

**Rules:**
1. Prerender everything possible (Astro 5.x is static by default, no `output` config needed)
2. Zero client-side JS by default — Astro's #1 advantage
3. No React/Vue/Svelte for simple UI — use `<script>` tags
4. `client:visible` over `client:load` for islands
5. No heavy npm packages (lodash, moment, axios) — use native APIs
6. CSS animations over JS animations
7. `<script is:inline>` for tiny scripts

**Banned patterns:** lodash, moment, axios, eval(), dynamic import(), Node.js fs/path/crypto

## Styling
- Design tokens in `src/styles/tokens.css`
- Component styles in Astro scoped `<style>` blocks
- Breakpoints: 768px (tablet), 1024px (desktop) — mobile-first
- Use CSS variables from tokens, not hardcoded values

## Cloudflare Workers
- Access runtime: `const { env, ctx } = Astro.locals.runtime;`
- After changing `wrangler.json` bindings, run `npm run cf-typegen`
- Static pages: default (prerendered at build)
- SSR pages: add `export const prerender = false;`
- Runtime secrets via `.dev.vars` locally, Cloudflare dashboard in prod

## Content
- Blog schema: title, description, pubDate (required), updatedDate, heroImage, tags, draft, author (optional)
- Filter drafts: `getCollection('blog', ({ data }) => !data.draft)`
- Dates in ISO 8601: `pubDate: 2026-03-21`
- Tags: lowercase kebab-case

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jarvis823)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jarvis823)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
