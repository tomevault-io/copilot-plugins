## kaos-web-aktuel

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Dev Commands

```bash
npm run dev      # Start dev server (localhost:3000)
npm run build    # Production build
npm run start    # Start production server
npm run lint     # ESLint
```

## Tech Stack

- **Next.js 16.1.6** (App Router, Turbopack)
- **React 19**, TypeScript 5.9
- **Tailwind CSS 4** via `@tailwindcss/postcss` – uses `@theme {}` block in globals.css for design tokens (not a tailwind.config.ts)
- **Framer Motion** for scroll animations
- **Font:** Inter (Google Fonts, weights 300–800)

## Architecture

### Routing

Pages live under `src/app/pages/<name>/page.tsx`. The homepage (`src/app/page.tsx`) is a one-pager composing all section components. Sub-pages use the shared `Layout` component from `src/layouts/Layout.tsx` which wraps content with Navbar + Footer + `pt-20` offset.

### Design System

Dark theme only. All colors defined as CSS custom properties in `src/styles/globals.css` inside `@theme {}`:
- Background: `#000000` / `#0a0a0a` (alt)
- Accent: `#cc0000` (red), hover `#ff0000`
- Text: `#ffffff` (primary), `#999999` (secondary), `#666666` (muted)

Use Tailwind classes like `bg-background`, `text-accent`, `border-border` – these map to the `@theme` tokens.

**Important:** Do NOT add a `* { margin: 0 }` CSS reset – it breaks Tailwind's `mx-auto`. Tailwind v4's `@import "tailwindcss"` includes its own preflight.

### Component Organization

- `src/components/ui/` – Shared atomic components: `Button`, `SectionWrapper`, `SectionHeading`, `Navbar`, `Footer`
- `src/components/sections/` – Homepage sections: `Hero`, `ServicesSection`, `AboutSection`, `PortfolioSection`, `BlogSection`, `ContactSection`
- `src/components/sections/BlogGrid.tsx` – Client Component (Framer Motion) für die Blog-Vollseite, bekommt `posts: BlogPost[]` als Props
- `src/layouts/` – Page wrappers (`Layout.tsx` is the standard one)

### API Routes

- `src/app/api/blog/route.ts` – Proxy zur Dev.to API. Liefert normalisierte Artikel (`title`, `excerpt`, `image`, `category`, `date`, `url`, `readingTime`). Query-Param: `?limit=N`. Cache: 1h (`revalidate: 3600`). Filtert Artikel ohne `cover_image` heraus. Exportiert auch den Typ `BlogPost`.

### Blog-Architektur

- `src/app/pages/blog/page.tsx` – **Server Component** (kein `'use client'`). Fetcht `/api/blog?limit=9` server-seitig und übergibt Daten an `BlogGrid`.
- `src/components/sections/BlogSection.tsx` – **Client Component**. Fetcht `/api/blog?limit=3` via `useEffect` on mount. Zeigt 3-Karten-Skeleton-Loading während des Ladens. Artikel verlinken auf Dev.to.
- Tags der Dev.to API: `marketing`, `webdev`, `design`, `seo`, `branding`

### Footer

- `src/components/ui/Footer.tsx` hat ein **Hintergrundvideo** (`/public/backgrounds/12882-242487537_medium.mp4`), das mit `autoPlay muted loop playsInline` läuft.
- Darüber liegt ein `bg-black/75` Overlay (als `absolute inset-0` div) für Lesbarkeit.
- Der gesamte Inhalt liegt in einem `relative` Container über dem Overlay.
- Video-Pattern: `<video>` als `absolute inset-0 w-full h-full object-cover`.

### Patterns

- All interactive components need `'use client'` directive
- Server Components (z.B. `blog/page.tsx`) sind `async` und fetchen direkt – kein `useEffect`
- Scroll animations use `whileInView={{ opacity: 1, y: 0 }}` with `viewport={{ once: true }}`
- Staggered lists: `transition={{ delay: i * 0.1 }}`
- `SectionWrapper` handles consistent padding, max-width, bg color alternation, and scroll-in animation
- Import alias: `@/*` maps to `./src/*`
- Content ist bei Blog-Seiten dynamisch via API, bei anderen Seiten hardcoded

### Image Configuration

`next.config.ts` verwendet `images.remotePatterns` (nicht das veraltete `domains`):
- `images.unsplash.com`, `cdn.pixabay.com`, `cdn.shopify.com`, `res.cloudinary.com`
- `**.dev.to` (Wildcard für alle Dev.to Subdomains: media.dev.to, media2.dev.to, etc.)
- `dev-to-uploads.s3.amazonaws.com`

### Build Gotchas

- `kaos-reset/` subfolder exists inside the project – excluded in `tsconfig.json` to prevent build conflicts
- `next.config.ts` nutzt jetzt `images.remotePatterns` statt `images.domains`
- Empty page files cause build errors – every `page.tsx` must have a default export
- Logo file: `/public/kaosLogo.jpeg` used in Navbar and Footer via next/image
- Nach Änderungen an `next.config.ts` muss der Dev-Server neu gestartet werden
- Dev.to liefert Bilder von verschiedenen Subdomains → Wildcard `**.dev.to` nötig, sonst `Invalid src prop` Fehler

---
> Source: [OGDeniz/kaos-web-aktuel](https://github.com/OGDeniz/kaos-web-aktuel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
