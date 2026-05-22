## unique-garden

> **Unique Garden Tech Cont. LLC** — A business website for a landscaping, garden design, irrigation, HVAC, and MEP services company operating in Dubai & Sharjah, UAE.

# CLAUDE.md — Developer Guide for Claude AI

## Project Overview

**Unique Garden Tech Cont. LLC** — A business website for a landscaping, garden design, irrigation, HVAC, and MEP services company operating in Dubai & Sharjah, UAE.

- **Framework:** Next.js 15.5 with App Router
- **Language:** TypeScript (strict mode)
- **Deployment:** Vercel (https://uniquegarden.vercel.app)

---

## Commands

```bash
npm run dev       # Start development server (http://localhost:3000)
npm run build     # Production build
npm run start     # Serve production build
npm run lint      # Run ESLint
```

---

## Project Structure

```
src/app/
├── components/        # Reusable React components
├── styles/            # Global and component SCSS/CSS
├── api/contact/       # Email form API route
└── pages/             # Route pages (App Router)
    ├── page.tsx           # Homepage
    ├── layout.tsx         # Root layout
    ├── about-us/
    ├── services/
    ├── projects/
    ├── gallery/
    ├── careers/
    ├── contact-us/
    ├── not-found.tsx
    └── sitemap.ts

public/
└── images/            # Organized by page (01-home-page/, 02-About-Us/, etc.)
```

Path alias: `@/*` maps to `./src/*`

---

## Architecture Patterns

### Client vs Server Components

- **Server components (default):** Static section components, Footer, page layouts
- **"use client" components:** Navbar (active links), ContactForm, MobileClassHandler, GlobalAnimations, anything using GSAP/Swiper/Lenis

### Page Composition Pattern

Pages are compositions of section components:

```tsx
export default function Page() {
  return (
    <>
      <HeroBanner ... />
      <SectionOne />
      <SectionTwo />
      <RequestAQuote />
    </>
  )
}
```

### Widget Wrapper Pattern

All major sections use `.widget-wrapper` as the outer class with optional `.bg` for background color:

```tsx
<section className="widget-wrapper bg our-services">
  <div className="text-content">...</div>
  <div className="image-content-wrapper">...</div>
</section>
```

### Animation System

Three layers of animation:
1. **Initial load:** CSS keyframes in `animation.scss` (navbar, hero text)
2. **Scroll-triggered:** GSAP ScrollTrigger in `GlobalAnimations.tsx`
3. **Smooth scroll:** Lenis via `<ReactLenis root />` in root layout

GSAP targets `.gsap-stagger-group` for staggered reveals and `.gsap-solo` for individual reveals. Always set `once: true` on scroll triggers (animations fire only once).

---

## Styling Rules

- **Tailwind CSS v4** for utility classes
- **SCSS** for component-specific and animation styles
- **No inline styles** — use class names
- Mobile styles use `.mobile` class on `<body>` (added by `MobileClassHandler`)
- Base viewport scale: 1920px — use `clamp()` for fluid sizing
- Mobile breakpoint: 767px

### Style File Map

```
src/app/styles/
├── globals.css             # CSS custom properties, Tailwind import
├── typography.css          # Font imports and CSS font variables
├── animation.scss          # GSAP + CSS keyframe animations
├── buttons.scss            # Button variants
├── scrollbar.scss          # Custom scrollbar
├── spacings.scss           # Margin/padding helpers
├── swiper.scss             # Swiper slider overrides
└── components/
    ├── all-components.scss # Imports all component styles
    ├── components.scss     # General component styles
    └── custom-pages.scss   # Page-specific overrides
```

---

## Key Libraries

| Library | Version | Use |
|---------|---------|-----|
| GSAP + @gsap/react | 3.14.2 | Scroll animations |
| Lenis + @studio-freight/react-lenis | 1.3.17 | Smooth scrolling |
| Swiper | 12.0.3 | Carousels and sliders |
| Nodemailer | 8.0.1 | SMTP email sending |
| react-google-recaptcha-v3 | 1.11.0 | Bot protection |

---

## Contact Form & API

`POST /api/contact` handles form submissions:

- **reCAPTCHA v3** score threshold: 0.5 (rejects below)
- **Honeypot fields:** `companyName` and `faxNumber` — must be empty
- Sends two emails: admin notification + customer confirmation
- SMTP via Gmail (Nodemailer)

Environment variables required (`.env.local`):
```
SMTP_HOST, SMTP_PORT, SMTP_USER, SMTP_PASS
ADMIN_EMAIL, REPLY_TO_EMAIL
NEXT_PUBLIC_RECAPTCHA_SITE_KEY, RECAPTCHA_SECRET_KEY
```

---

## SEO

- Metadata defined per page using Next.js `generateMetadata` or static `metadata` export
- Schema.org `LocalBusiness` JSON-LD on homepage
- OpenGraph + Twitter Card images per page
- `sitemap.ts` generates XML sitemap dynamically

---

## TypeScript

- Strict mode enabled
- No `any` types unless unavoidable
- Use proper interfaces for component props
- GSAP refs typed as `gsap.core.Timeline | null`

---

## Do Not Do

- Do not add unnecessary client-side libraries — prefer SSR where possible
- Do not hardcode colors — use CSS variables defined in `globals.css`
- Do not skip reCAPTCHA validation on the contact API route
- Do not commit `.env.local`
- Do not use `export default` on server action files (use named exports)

---
> Source: [gowthamavtech/unique-garden](https://github.com/gowthamavtech/unique-garden) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
