## myclub-fanpost

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**KANVA** is a social media post generator for Swiss sports clubs (Unihockey, Volleyball, and Handball). It automatically creates Instagram-ready posts from sports game data, positioned as "Canva for Sports".

**Tech Stack:**
- React 18 + TypeScript + Vite
- Supabase (PostgreSQL, Auth, Storage, Edge Functions)
- Shadcn/ui components on Tailwind CSS
- TanStack Query for data fetching
- Stripe for subscriptions

## Development Commands

```bash
# Install dependencies
npm install

# Start dev server (runs on http://[::]:8080)
npm run dev

# Build for production
npm run build

# Build for development (with source maps)
npm run build:dev

# Lint code
npm run lint

# Preview production build
npm run preview
```

**Important:** There are no test scripts configured. TypeScript is configured with relaxed rules (`noImplicitAny: false`, `strictNullChecks: false`).

## Architecture

### Core Flow
```
User selects: Sport → Club → Team → Game(s)
  ↓
External Sports APIs (Cloud Functions with GraphQL-like queries)
  ↓
React Query cache
  ↓
Template selection & customization
  ↓
SVG generation with game data
  ↓
Export to PNG (Instagram-optimized: 2160x2700px)
```

### Key Directories

```
src/
├── components/
│   ├── ui/              # Shadcn components (51 files)
│   ├── landing/         # Landing page sections
│   ├── profile/         # User profile & settings
│   └── templates/       # Template designer & management
├── pages/               # Route pages (Index, Studio, Profile, etc.)
├── hooks/               # Custom hooks (useAuth, useSubscription, etc.)
├── contexts/            # React Context providers
├── utils/               # Core utilities (SVG processing)
├── config/              # Configuration (fonts, etc.)
├── integrations/
│   └── supabase/        # Supabase client & generated types
├── translations/        # i18n support (DE/EN)
└── assets/

supabase/
├── functions/           # Edge functions (7 functions)
└── migrations/          # Database migrations (47 files)
```

### Critical Components

- **Studio.tsx** (656 lines): Main app - game selection & post generation
- **GamePreviewDisplay.tsx** (1150+ lines): SVG template preview & PNG export
- **TemplateDesigner.tsx** (2280+ lines): Visual template editor

### Sports Data APIs

Game data comes from **external Cloud Functions**, NOT Supabase:
- **Unihockey**: `myclubmanagement.cloudfunctions.net/api/swissunihockey`
- **Volleyball**: `myclubmanagement.cloudfunctions.net/api/swissvolley`
- **Handball**: `myclubmanagement.cloudfunctions.net/api/swisshandball`

These use GraphQL-like query interfaces.

### Database Schema (Key Tables)

From `src/integrations/supabase/types.ts`:
- **profiles**: User profiles, email preferences, last selections
- **templates**: SVG template configurations (system & custom)
- **user_logos**: Uploaded team/club/sponsor logos
- **user_team_slots**: Teams user follows for notifications
- **subscriptions**: Stripe subscription data
- **subscription_limits**: Feature limits per tier (Free, Amateur, Pro, Premium)

## SVG to PNG Export System

This is the most complex part of the application. **READ THIS CAREFULLY** before modifying export code.

### Two Export Implementations

**1. Legacy: `src/utils/svgToImage.ts` (976 lines)**
- Monolithic approach
- Processes entire SVG at once
- Uses html2canvas for rendering
- Still available as fallback

**2. New (Preferred): `src/utils/svgToImageLayered.ts` (1516 lines)**
- **60% faster, 60% less memory usage, 90% more reliable**
- Separates SVG into 3 layers:
  - Background layer (colors/background images)
  - Images layer (team logos, graphics)
  - Text layer (text + embedded fonts)
- Uses native `createImageBitmap()` instead of html2canvas
- Better font rendering reliability

See `docs/layered-svg-export.md` for full technical documentation.

### Export Process Phases

1. **SVG Preparation**: Clone and offscreen mount SVG
2. **Image Inlining** (20-60%): Convert HTTP URLs to data URLs
   - Uses CORS fetch with Supabase proxy fallback
   - Progress tracking per image
3. **Font Management** (60-70%): Embed custom fonts
   - Fonts configured in `src/config/fonts.ts` (Bebas Neue, Roboto, Open Sans, Lato, Montserrat)
   - WOFF2 format converted to data URLs
   - Three embedding methods for maximum compatibility
4. **Canvas Rendering** (70-90%): Convert layers to canvas
5. **Finalization** (90-100%): Export as PNG/JPEG/WebP

### Font Handling

All fonts are centrally configured in `src/config/fonts.ts`:
```typescript
export const AVAILABLE_FONTS: Record<string, FontConfig> = {
  'bebas-neue': {
    displayName: 'Bebas Neue',
    cssFamily: 'Bebas Neue',
    googleFontsUrl: '...',
    variants: [{ weight: '400', style: 'normal', url: '...' }]
  },
  // ... more fonts
}
```

**Important:** Font names must match exactly between SVG `font-family` attributes and `cssFamily` in config.

### Platform-Specific Export

- **Mobile**: Uses native Share API (Capacitor)
- **Desktop**: Direct download
- **Instagram optimization**: 1080x1350 base @ 2x scale = 2160x2700px

## Subscription & Limits

Subscription tiers gate features via `useSubscriptionLimits()` hook:
- **Free**: Basic features, limited templates
- **Amateur**: Email notifications, more templates
- **Pro**: Logo uploads, sponsor integration
- **Premium**: Up to 15 team slots, all features

Stripe price IDs are hardcoded in `src/hooks/useSubscription.tsx`.

## Supabase Edge Functions

Located in `supabase/functions/`:
- **check-subscription**: Verify Stripe subscription status
- **create-checkout**: Create Stripe checkout session
- **customer-portal**: Open Stripe customer portal
- **delete-account**: GDPR-compliant account deletion
- **image-proxy**: CORS proxy for external images
- **send-game-reminders**: Cron job for game day emails
- **send-game-announcements**: Cron job for upcoming game posts

## Authentication

Uses Supabase Auth with **magic link** (passwordless email authentication).

## Routing & Deep Linking

React Router with deep linking support:
- URL structure: `/studio/:sport/:clubId/:teamId/:gameId`
- Multiple game IDs supported in URL
- All routes configured in `src/App.tsx`
- Vercel SPA fallback in `vercel.json`

## Internationalization

Two languages supported: German (DE) and English (EN)
- Translations in `src/translations/`
- `useLanguage()` hook for access
- **Known issue:** Profile page missing English translations (GitHub issue #2)

## Brand Guidelines

From `designkonzept.md`:
- **Colors**: Electric Blue (#2979FF), Vivid Coral (#FF4E56), Midnight Slate (#1C1C28)
- **Typography**: Satoshi Bold (headlines), Manrope (body), Space Grotesk (scores)
- **Mission**: "Canva for Sports" - simple, emotional, team-driven
- **Claim**: "Design your win."

## Important Notes

### Code Organization
- Large component files exist (1000+ lines) - be careful when editing
- Prefer reading full context before suggesting changes
- TypeScript is relaxed - types may not catch all errors

### Areas to Watch
- **SVG Export**: Complex, performance-sensitive - test thoroughly
- **Font Rendering**: Multiple fallback strategies - don't simplify without testing
- **CORS Handling**: Image proxy is critical for external images
- **Subscription Limits**: Check `useSubscriptionLimits()` before adding features

### Development Practices
- No tests configured - manual testing required
- ESLint warns but doesn't enforce strictly
- Bun lockfile present (alternative to npm)
- Path alias `@` maps to `./src`

### When Modifying Export Code
1. Read `docs/layered-svg-export.md` first
2. Test with multiple templates (single-game and multi-game)
3. Test with different fonts and images
4. Test on mobile (Share API) and desktop (download)
5. Monitor browser console for font loading errors
6. Check exported image quality at 2x scale (2160x2700px)

### Mobile Support
- Capacitor 7 configured for iOS/Android
- Native filesystem and share APIs available
- Test exports on mobile devices if modifying export code

## Common Workflows

### Adding a New Template
1. Create SVG template in TemplateDesigner
2. Save to `templates` table in Supabase
3. Configure which subscription tier can access it
4. Test export with real game data

### Adding a New Font
1. Add font config to `src/config/fonts.ts`
2. Include Google Fonts URL and WOFF2 direct URL
3. Test font rendering in SVG preview
4. Test font embedding in PNG export

### Debugging Export Issues
1. Check browser console for font loading errors
2. Use layer inspection (see `docs/layered-svg-export.md`)
3. Verify CORS headers for external images
4. Test with image proxy if direct fetch fails
5. Check `fontCache` and `imageDataUrlCache` in code

### Adding a New Sport
1. Add Cloud Function endpoint URL to code
2. Update sport type in TypeScript types
3. Add sport-specific UI elements
4. Test game data fetching and display

---
> Source: [myclubapp/myclub-fanpost](https://github.com/myclubapp/myclub-fanpost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
