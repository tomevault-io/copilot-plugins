## novamir

> - React 19 + TypeScript + Vite

# Novamir Project

## Tech Stack
- React 19 + TypeScript + Vite
- TailwindCSS + Framer Motion
- Deployed on Vercel

## Key Files
- `index.html` — SPA shell, meta tags, JSON-LD structured data, SEO fallback content
- `public/*.html` — Static pages (about, contact, privacy-policy, terms-conditions)
- `components/*.tsx` — React components (Hero, ProductSection, Footer, Header, etc.)
- `components/ui/*.tsx` — shadcn/ui components (timeline, footer-section, animated-tabs)
- `assets/` — Images and logos

## Commands
- `npm run dev` — Start dev server
- `npm run build` — Production build (Vite)
- `squirrel audit https://novamir.com --format llm` — Run SEO audit

## Architecture
- This is a SPA with pre-rendered static pages in /public
- The main page meta tags and structured data are in index.html
- index.html contains an SEO fallback (`<header>`, `<main>`, `<footer>`) that is visually hidden but parsed by crawlers
- Static pages (about, contact, privacy-policy, terms-conditions) have their own full HTML — they are NOT React
- Hero.tsx renders the visible H1 for the homepage
- The Footer component is in `components/ui/footer-section.tsx`

## SEO Notes
- JSON-LD structured data is in `<script type="application/ld+json">` blocks in the `<head>` of index.html
- Each public page has its own JSON-LD (WebPage, AboutPage, ContactPage schemas)
- All pages have Open Graph and Twitter Card meta tags
- sitemap.xml and robots.txt are in /public

## Important Constraints
- Do NOT change visual design or functionality — only fix SEO/technical issues
- Do NOT modify Tailwind config, Vite config, or package dependencies
- When editing public/*.html, maintain the existing styling and layout
- The SEO fallback in index.html uses `clip: rect(0,0,0,0)` to hide from visual users — do not remove this

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EmPi44) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
