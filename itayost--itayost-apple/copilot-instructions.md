## itayost-apple

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Next.js 14 (App Router) portfolio/business website with Apple-inspired design, Hebrew RTL support, and advanced animations. Built with TypeScript, Tailwind CSS, Framer Motion, and GSAP.

## Development Commands

```bash
# Development
npm run dev              # Start dev server on http://localhost:3000

# Type Checking & Linting
npm run type-check       # TypeScript type checking (no emit)
npm run lint             # Run ESLint
npm run format           # Format code with Prettier

# Build & Production
npm run build            # Build for production
npm run start            # Start production server
npm run analyze          # Build with bundle size analysis
```

## Architecture

### Page Structure Pattern

All routes follow a two-file pattern:
- `app/[route]/page.tsx` - Minimal Next.js page wrapper with metadata
- `app/[route]/[Route]Page.tsx` - Actual page component with content

Example:
```
app/services/
├── page.tsx          # Export metadata + render ServicesPage
└── ServicesPage.tsx  # Full page component
```

This separation keeps metadata configuration clean and allows for better code organization.

### Configuration-Driven Content

All content is centralized in `/src/config/`:
- `content.ts` - All Hebrew text content (14KB+ of structured content)
- `seo.ts` - SEO metadata, structured data (JSON-LD), Open Graph tags
- `socialLinks.ts` - Social media links
- `fonts.ts` - Font configurations

**When editing content**: Always update `src/config/content.ts` rather than hardcoding strings in components.

### Right-to-Left (RTL) Hebrew Architecture

The entire site is RTL-first:
- Root layout sets `dir="rtl"` and `lang="he"` on `<html>`
- Tailwind configured with logical properties for RTL support
- Heebo font loaded with Hebrew subset prioritized
- All components use RTL-aware spacing and layout

**Critical**: When adding UI components, test RTL layout. Use logical properties (`start`/`end`) instead of `left`/`right`.

### Animation System

Dual animation system for Apple-style effects:

1. **Framer Motion** - Component-level animations
   - Used in buttons, cards, modal transitions
   - Handles interactive animations and layout animations

2. **GSAP** - Scroll-based animations
   - Advanced scroll effects via `useAppleScrollEffects` hook
   - Used in `ScrollAnimations/` components (AppleParallax, SmoothReveal, TextReveal, etc.)

**Components in `/src/components/ScrollAnimations/`**:
- `AppleParallax.tsx` - Parallax scrolling effects
- `SmoothReveal.tsx` - Fade-in on scroll with direction control
- `TextReveal.tsx` - Character-by-character text reveals
- `StickyScale.tsx` - Scale effects on sticky elements
- `AppleAnimations.tsx` - Composite animation patterns

**Key Hook**: `useAppleScrollEffects` tracks scroll position, direction, and scrolling state for complex scroll-linked animations.

### Component Organization

```
components/
├── ScrollAnimations/   # GSAP scroll effects
├── layout/            # Navigation & Footer
├── sections/          # Large page sections (reusable across pages)
├── common/           # Shared utilities (WhatsAppButton, JsonLd, WebVitals)
└── ui/              # Design system components (Button, Card, Input, etc.)
```

### SEO & Performance Optimization

**SEO Strategy**:
- Complete metadata API usage in root layout
- Structured data (JSON-LD) for Organization, Website, LocalBusiness
- Dynamic sitemap generation (`app/sitemap.ts`)
- Robots.txt configuration (`app/robots.ts`)
- Canonical URLs and language alternates
- Open Graph and Twitter Card tags

**Performance Optimizations**:
- Image optimization with Sharp (AVIF/WebP formats)
- Font preloading with display: swap
- Preconnect to external domains (Google Fonts, Analytics)
- Code splitting with dynamic imports
- Console.log removal in production
- Security headers in `next.config.js`
- Aggressive caching for static assets

**Bundle Analysis**: Run `npm run analyze` to view bundle size breakdown.

### Custom Hooks

Located in `/src/hooks/`:
- `useAppleScrollEffects.ts` - Scroll position, direction, and state tracking
- `useIntersectionObserver.ts` - Viewport detection for lazy loading
- `useParallax.ts` - Parallax offset calculations
- `useScrollProgress.ts` - Scroll progress percentage
- `usePerformance.ts` - Web Vitals monitoring
- `useWindowSize.ts` - Responsive breakpoint detection

These hooks power the animation system and performance monitoring throughout the site.

## Key Technical Details

### Heebo Font Configuration

The site uses Heebo font with Hebrew and Latin subsets. Font loading is optimized with:
- `display: 'swap'` to prevent render blocking
- Preloading for critical font files
- Only necessary weights loaded (400, 500, 600, 700)
- System font fallback chain

### Apple Color System

Tailwind extended with Apple design colors:
- Primary: `apple-blue` (#0071E3)
- Secondary: `apple-purple` (#BF5AF2)
- Accents: `apple-pink`, `apple-green`, `apple-cyan`
- Gray scale: `apple-gray-{50-900}` matching Apple's design system

Use these color names in Tailwind classes for consistency with the design language.

### Analytics

**GA4**: 7 custom events defined in `src/lib/analytics.ts`, typed in `src/types/analytics.ts`:
- `generate_lead`, `whatsapp_click`, `contact_click`, `cta_click`, `service_view`, `portfolio_click`, `outbound_click`
- Loaded in `src/app/layout.tsx` via `GoogleAnalytics` component. `window.gtag` must be assigned to `window` (not declared as local function) for cross-module access.

**PostHog** (product analytics with autocapture):
- Initialized via `instrumentation-client.ts` at project root (Next.js 15.3+ pattern)
- Autocapture handles pageviews, clicks, form interactions automatically
- Server-side lead tracking in `/api/leads` via `posthog-node` (`src/lib/posthog-server.ts`)
- EU region: `https://eu.i.posthog.com`
- Client-side access: `import posthog from 'posthog-js'`

**Also active**: Vercel Analytics, Vercel Speed Insights, Microsoft Clarity.

Contact form (`src/app/contact/ContactPage.tsx`) submits to `/api/leads` which proxies to external CRM. Track analytics before API calls, not after success callbacks.

### WhatsApp Integration

Floating WhatsApp button component (`WhatsAppButton`) is globally rendered in root layout. Contact info is centralized in config. The button uses:
- Fixed positioning with mobile-optimized placement
- Hebrew-formatted phone number (054-499-4417)
- Direct wa.me link integration

### Environment Variables

Required for production:
```
NEXT_PUBLIC_GA_ID=G-XXXXXXXXXX
NEXT_PUBLIC_SITE_URL=https://www.itayost.com
NEXT_PUBLIC_POSTHOG_KEY=phc_xxx
NEXT_PUBLIC_POSTHOG_HOST=https://eu.i.posthog.com
```

Optional:
```
NEXT_PUBLIC_GOOGLE_SITE_VERIFICATION=
NEXT_PUBLIC_BING_SITE_VERIFICATION=
```

Google Reviews integration (optional - uses fallback data if not configured):
```
GOOGLE_PLACES_API_KEY=your-google-places-api-key
GOOGLE_PLACE_ID=your-google-place-id
```

To get these values:
1. Enable the Places API in Google Cloud Console
2. Create an API key with Places API access
3. Find your Place ID at: https://developers.google.com/maps/documentation/places/web-service/place-id

## Working with This Codebase

### Adding New Pages

1. Create folder in `src/app/your-route/`
2. Add `page.tsx` (metadata + wrapper)
3. Add `YourRoutePage.tsx` (actual component)
4. Update navigation in `components/layout/Navigation.tsx`
5. Add SEO config in `config/seo.ts`
6. Test RTL layout and mobile responsiveness

### Modifying Content

- Hebrew content: Edit `src/config/content.ts`
- SEO metadata: Edit `src/config/seo.ts`
- Social links: Edit `src/config/socialLinks.ts`
- Portfolio items: Edit `src/data/portfolio.ts`

### Adding Animations

For scroll animations, use existing ScrollAnimations components or create new ones following the pattern:
1. Use `useAppleScrollEffects` hook for scroll state
2. Implement with Framer Motion for smooth transitions
3. Consider performance - use `will-change` and `transform` for GPU acceleration
4. Test on mobile devices for performance

### RTL Development Notes

- Always use Tailwind's logical properties where possible
- Test all new components with RTL layout
- Icons and images should be flipped appropriately for RTL
- Text alignment defaults to right in RTL context
- Flexbox and Grid reverse in RTL automatically

## Build & Deployment

The site is optimized for Vercel deployment. Auto-deploys on push to `main`.
- Vercel Project: `prj_RMH49pnUO6ezRZfJ5xk1nS3AMlXH`
- Vercel Team: `team_L6vq0d4dnBw36bBjG5EagW7H`

Build process:
1. Type checking (`tsc --noEmit`)
2. Next.js build with optimizations
3. Static asset optimization
4. Bundle size analysis available via `npm run analyze`

Production builds automatically:
- Remove console.log statements
- Optimize images to AVIF/WebP
- Generate sitemap and robots.txt
- Apply security headers

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/itayost)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/itayost)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
