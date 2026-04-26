## asimov

> <!-- GSD:project-start source:PROJECT.md -->

<!-- GSD:project-start source:PROJECT.md -->
## Project

**Asimov Atlas**

An interactive, bilingual (EN/CZ) website that transforms Isaac Asimov's Robots/Empire/Foundation reading order flowchart into an animated, explorable timeline. Visitors discover each book through series-themed detail cards with synopses, ratings, cover art, and purchase links. Built with Next.js 15 SSG, GSAP, and tsParticles, hosted on Cloudflare Pages.

**Core Value:** The interactive timeline IS the product — if it doesn't feel amazing to explore, nothing else matters.

### Constraints

- **Tech stack**: Next.js 15 (App Router, SSG), TypeScript, Tailwind CSS v4, shadcn/ui, Magic UI + Aceternity UI, GSAP 3 + ScrollTrigger, Framer Motion 11, tsParticles 3, next-intl, Zustand 5
- **Hosting**: Cloudflare Pages (edge CDN, free tier, auto-deploy via `@cloudflare/next-on-pages`)
- **Performance**: LCP <2.5s mobile, INP <200ms, animation >55fps mobile, JS bundle <150KB gzipped, Lighthouse >95
- **Data**: Static TypeScript files compiled at build time — no database, no API runtime dependency
- **Images**: Cover art from Open Library API at build time, fallback to Cloudflare R2; fair use monitored
- **Analytics**: Cloudflare Web Analytics (privacy-first, no cookie banner)
<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->
## Technology Stack

## Critical Discovery: Deployment Path
| Path | Approach | Complexity | Trade-offs |
|------|----------|------------|------------|
| **A: Pure Static Export** | `output: 'export'` in next.config, deploy HTML/CSS/JS to Cloudflare Pages | Low | Simplest; loses ISR, middleware, server actions. Fine for this project since all data is static. |
| **B: OpenNext + Workers** | `@opennextjs/cloudflare` adapter, deploy to Cloudflare Workers | Medium | Full Next.js feature set; more complex build pipeline; Workers has compute limits. |
## Recommended Stack
### Core Framework
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| Next.js | 15.5.x | Framework, SSG, routing | Stable, well-tested on Cloudflare Pages static export. v16 exists but v15 is battle-tested for static export and has broader ecosystem compatibility. | HIGH |
| React | 19.x | UI library | Ships with Next.js 15; concurrent features, compiler optimizations | HIGH |
| TypeScript | 5.7+ | Type safety | Non-negotiable for a data-heavy project with 40+ book objects | HIGH |
### Styling
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| Tailwind CSS | 4.x | Utility-first CSS | v4 is stable (released Jan 2025), CSS-first config with @theme, 5x faster builds. Series-themed CSS custom properties map perfectly to Tailwind v4's @theme directive. | HIGH |
| CSS Custom Properties | -- | Series theming | Robot=steel-blue, Empire=gold, Foundation=purple via CSS vars. Zero-JS theme switching. | HIGH |
### UI Components
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| shadcn/ui | CLI v4 | Base components (dialog, command palette, cards) | Copy-paste components, not a dependency. Works with Tailwind v4 and Radix primitives. The Cmd+K command palette comes from shadcn's `<Command>` component (built on cmdk). | HIGH |
| Aceternity UI | latest | Timeline visual effects, glowing cards, tracing beams | Copy-paste animated components built on Framer Motion + Tailwind. Perfect for the series-themed book cards and cross-series connection visualizations. | MEDIUM |
| Magic UI | latest | Hero section, star-field overlays, marketing polish | Complements shadcn for landing/marketing elements. "Series A aesthetic" without custom animation work. | MEDIUM |
### Animation
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| GSAP | 3.14.x | Timeline animation, scroll-driven layouts, reading-order re-layout | Industry-standard animation. Now 100% free (Webflow acquired GreenSock, open-sourced everything including ScrollTrigger, SplitText, MorphSVG). ~25KB gzipped. | HIGH |
| GSAP ScrollTrigger | (bundled) | Scroll-driven animation triggers | Bundled with GSAP. Controls era-specific color shifts, timeline node reveals on scroll. | HIGH |
| Motion (formerly Framer Motion) | 12.x | Component enter/exit, layout animations, gesture support | Renamed from `framer-motion` to `motion`. Import from `motion/react`. Use for React-native animations (mount/unmount, layout shifts, drag). GSAP handles the timeline; Motion handles component transitions. | HIGH |
- **GSAP:** Timeline scroll animation, ScrollTrigger-driven reveals, reading-order-to-chronological re-layout animation, star-field color transitions
- **Motion:** Card enter/exit, hover previews, language toggle transitions, command palette open/close, layout animations via `layoutId`
### Particles
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| @tsparticles/react | 3.0.0 | React wrapper for tsParticles | Official React component for tsParticles v3. | MEDIUM |
| @tsparticles/slim | 3.x | Particle engine (lightweight bundle) | Use `slim` not `full` -- includes stars, links, and basic shapes without heavy plugins. Keeps bundle small for the star-field background. | MEDIUM |
### Internationalization
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| next-intl | 4.8.x | i18n routing, message formatting, locale switching | App Router native, URL-based locale (`/en/`, `/cs/`), works with `output: 'export'` via `generateStaticParams`, ICU message syntax, type-safe. | HIGH |
### State Management
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| Zustand | 5.0.x | Client state (animation tier, active filters, view mode toggle) | Zero boilerplate, 1KB, no providers needed. Perfect for "reading order vs chronological" toggle, animation tier preference, active series filter. | HIGH |
### Search
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| cmdk | 1.x | Command palette UI | Powers shadcn's `<Command>` component. Accessible, composable, keyboard-first. | HIGH |
| Fuse.js | 7.x | Fuzzy search over bilingual book data | Client-side fuzzy search. Index both EN and CZ titles/synopses at build time. ~5KB. | HIGH |
### SEO & Meta
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| Next.js Metadata API | (built-in) | JSON-LD, OG tags, sitemap | Built into Next.js 15. `generateMetadata` + `generateStaticParams` for per-book SEO. | HIGH |
| @vercel/og or satori | latest | OG image generation | Generate OG images at build time per book. Since static export, generate during build and output as static images. | MEDIUM |
| next-sitemap | 4.x | Sitemap with hreflang | Generates sitemap.xml with hreflang alternates for EN/CZ pages. | HIGH |
### Hosting & Infrastructure
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| Cloudflare Pages | -- | Static hosting, edge CDN | Free tier, auto-deploy from GitHub, global edge CDN. Perfect for static export. | HIGH |
| Cloudflare Web Analytics | -- | Privacy-first analytics | No cookies, no banner needed. Script tag, zero config. | HIGH |
| Cloudflare R2 | -- | Image fallback storage | If Open Library API covers are unavailable, store fallbacks in R2. | LOW (may not be needed) |
### Dev Tooling
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| ESLint | 9.x | Linting | Flat config, Next.js plugin | HIGH |
| Prettier | 3.x | Formatting | With prettier-plugin-tailwindcss for class sorting | HIGH |
| Turbopack | (built-in) | Dev server | Included in Next.js 15. ~10x faster HMR vs Webpack. | HIGH |
## Alternatives Considered
| Category | Recommended | Alternative | Why Not |
|----------|-------------|-------------|---------|
| Framework | Next.js 15 | Astro 5 | Astro's island architecture fights full-page interactivity. Every page needs GSAP timeline + particles. |
| Framework | Next.js 15 | Next.js 16 | v16 is newer but adds no value for static export; v15 is battle-tested |
| Deployment | Static Export + CF Pages | OpenNext + CF Workers | No server-side features needed; static is simpler and cheaper |
| Deployment | Static Export + CF Pages | @cloudflare/next-on-pages | **Deprecated.** Does not support Next.js 15. Do not use. |
| Animation | GSAP 3 | Anime.js | GSAP has superior ScrollTrigger, timeline sequencing, and ecosystem. Now free. |
| Animation | Motion 12 | Framer Motion 11 | Same library, renamed. Use `motion` package, import from `motion/react`. |
| Particles | tsParticles slim | Three.js | ADR-001 decided 2D. Three.js is 150KB+ vs tsParticles slim ~30KB. |
| i18n | next-intl 4 | next-translate | next-intl has better App Router support, type safety, and static export compat |
| State | Zustand 5 | Jotai | Either works; Zustand's store pattern is simpler for this use case (few stores, no atom graph) |
| Search | Fuse.js | Algolia | 40 books. Client-side fuzzy search is fine. Algolia is overkill and adds a dependency. |
| CSS | Tailwind v4 | Tailwind v3 | v4 is stable, faster, CSS-first config aligns with the series theming approach |
## Installation
# Core framework
# Styling
# Animation
# Particles
# i18n
# State
# Search
# SEO
# Dev dependencies
## Package.json Scripts
## next.config.ts Key Settings
## Version Summary
| Package | Version | Last Verified |
|---------|---------|---------------|
| next | 15.5.x | 2026-04-03 |
| react | 19.x | 2026-04-03 |
| tailwindcss | 4.x | 2026-04-03 |
| gsap | 3.14.x | 2026-04-03 |
| motion | 12.x | 2026-04-03 |
| @tsparticles/react | 3.0.0 | 2026-04-03 |
| next-intl | 4.8.x | 2026-04-03 |
| zustand | 5.0.x | 2026-04-03 |
| cmdk | 1.x | 2026-04-03 |
| fuse.js | 7.x | 2026-04-03 |
## Sources
- [Next.js Blog](https://nextjs.org/blog) -- version history and release notes
- [Next.js Static Export on Cloudflare Pages](https://developers.cloudflare.com/pages/framework-guides/nextjs/deploy-a-static-nextjs-site/) -- official deployment guide
- [OpenNext Cloudflare](https://opennext.js.org/cloudflare) -- alternative deployment path (not recommended for this project)
- [@cloudflare/next-on-pages deprecation](https://github.com/cloudflare/next-on-pages) -- deprecated, do not use
- [GSAP npm](https://www.npmjs.com/package/gsap) -- v3.14.2, now free
- [GSAP Licensing](https://gsap.com/licensing/) -- 100% free including all plugins
- [Motion (formerly Framer Motion)](https://motion.dev/) -- renamed, v12.x
- [tsParticles npm](https://www.npmjs.com/package/tsparticles) -- v3.9.1
- [next-intl](https://next-intl.dev/) -- v4.8.x, App Router native
- [Zustand npm](https://www.npmjs.com/package/zustand) -- v5.0.12
- [shadcn/ui Changelog](https://ui.shadcn.com/docs/changelog) -- CLI v4
- [Tailwind CSS v4](https://tailwindcss.com/blog/tailwindcss-v4) -- stable Jan 2025
- [Aceternity UI](https://ui.aceternity.com/) -- copy-paste animated components
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

Conventions not yet established. Will populate as patterns emerge during development.
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Architecture

Architecture not yet mapped. Follow existing patterns found in the codebase.
<!-- GSD:architecture-end -->

<!-- GSD:workflow-start source:GSD defaults -->
## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:
- `/gsd:quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd:debug` for investigation and bug fixing
- `/gsd:execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->



<!-- GSD:profile-start -->
## Developer Profile

> Profile not yet configured. Run `/gsd:profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/eletrixa) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
