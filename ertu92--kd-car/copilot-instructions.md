## kd-car

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

KD-CAR is a modern, responsive car detailing and sales website built with Next.js 14, React 18, TypeScript, and Tailwind CSS. The site serves a German car detailing business, featuring services showcase, vehicle inventory, contact forms, and email integration.

## Development Commands

- `npm run dev` - Start development server on http://localhost:3000
- `npm run build` - Build production bundle
- `npm start` - Start production server
- `npm run lint` - Run ESLint

## Architecture

### Next.js App Router Structure
- **app/**: App Router pages and API routes
  - `layout.tsx` - Root layout with Header, Footer, and CookieConsent
  - `page.tsx` - Home page assembling HeroSection, ServicesSection, AboutSection, ContactSection
  - `app/api/contact/route.ts` - Contact form email handler using nodemailer with Gmail
  - `app/inventory/page.tsx` - Vehicle inventory page
  - `app/impressum/page.tsx` - Legal imprint page (German requirement)

### Components Architecture
All components in `components/` are client-side React components using Framer Motion for animations:
- **Header.tsx** - Fixed navigation with scroll-based styling changes and section navigation
- **HeroSection.tsx** - Video background hero with supercar footage
- **ServicesSection.tsx** - Service cards for car detailing services
- **AboutSection.tsx** - Company information
- **ContactSection.tsx** - Contact form with client-side validation
- **InventorySection.tsx** - Vehicle listing with CarCard and CarDetailModal
- **Footer.tsx** - Site footer with links
- **CookieConsent.tsx** - GDPR cookie consent banner

### Styling System
Tailwind CSS with custom theme in `tailwind.config.js`:
- Primary colors: Sky blue scale (#0ea5e9)
- Accent colors: Yellow scale (#eab308)
- Dark colors: Slate scale
- Custom animations: fade-in, slide-up, float
- Responsive breakpoints: mobile-first design

### Email Configuration
Contact form sends emails via nodemailer + Gmail SMTP. Environment variables required in `.env.local`:
- `EMAIL_USER` - Gmail account
- `EMAIL_PASS` - Gmail app password (requires 2FA enabled)

Email setup details in EMAIL_SETUP.md. The API route at `app/api/contact/route.ts` handles form submissions with comprehensive error logging.

### Path Aliases
TypeScript configured with `@/*` alias mapping to root directory for clean imports:
```typescript
import Header from '@/components/Header'
```

### Image Handling
Next.js Image component configured to allow external domains:
- images.unsplash.com
- picsum.photos

Images primarily sourced from Unsplash URLs for service sections.

## Key Technical Patterns

### Navigation System
Header component uses `useRouter` for programmatic navigation. Navigation buttons check current pathname:
- If on homepage (`/`): scroll to section via `scrollIntoView`
- If on other page: navigate to `/#sectionId` then scroll

### State Management
No global state management. Components use local React state with hooks. Framer Motion handles animation states.

### Form Handling
Contact form in ContactSection.tsx:
1. Client-side validation
2. POST to `/api/contact`
3. Success/error feedback via state
4. Email sent to `info@kd-car.de`

### Responsive Design
Mobile-first Tailwind classes with breakpoints:
- Mobile: Default styles
- Tablet: `md:` prefix (768px+)
- Desktop: `lg:` prefix (1024px+)

## German Language Context
All UI text is in German. This is intentional as the target market is Germany. Keep German for:
- User-facing content
- Error messages
- SEO metadata
- Email templates

## Dependencies
Core: next, react, react-dom, typescript
Styling: tailwindcss, postcss, autoprefixer
Animation: framer-motion
Icons: lucide-react
Email: nodemailer, @types/nodemailer

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ertu92)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ertu92)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
