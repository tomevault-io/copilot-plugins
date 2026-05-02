## odio-custom-builders

> Construction company website with focus on:

# Odio Custom Builders - Development Guide

## Project Overview

Construction company website with focus on:
1. **Home repairs as gateway to renovations** - 70% of major projects come from past repair clients
2. **Realtor partner program** - Dedicated landing page and closing repair services
3. **Premium design** - NetJets-inspired minimalism with navy/gold palette

## Tech Stack

- **Framework**: Next.js 15 (App Router)
- **Styling**: Tailwind CSS v4 with custom design tokens
- **Deployment**: Vercel
- **Backend**: Supabase (for contact form submissions - pending setup)

## Key Patterns

### Design Tokens
All colors are defined in `src/app/globals.css` under `@theme`:
- Navy palette: `navy-950` through `navy-500`
- Gold palette: `gold-600` through `gold-100`
- Use semantic colors for consistency

### Component Architecture
```
src/components/
├── ui/          # Primitives (Button, Card, Input, etc.)
├── layout/      # Header, Footer
├── sections/    # Page sections (Hero, ServicesGrid, etc.)
└── forms/       # Form components (pending)
```

### Page Structure
All pages follow this pattern:
1. Hero section with background image
2. Content sections using `<Section>` component
3. CTA section at bottom

### Constants
Business info lives in `src/lib/constants.ts`:
- `SITE_CONFIG` - Contact info, hours, social links
- `NAV_ITEMS` - Navigation structure
- `SERVICES` - Service definitions
- Update these when deploying for a real client

## Deployment

### Vercel
1. Connect repo to Vercel
2. Framework preset: Next.js (auto-detected)
3. Build command: `npm run build`
4. Output directory: `.next`

### Environment Variables
Copy `.env.example` to `.env.local` and fill in:
- Supabase credentials (for contact forms)
- Email service credentials (for notifications)

## Future Enhancements

### Phase 2: Contact Form Backend
- Set up Supabase project
- Create `contact_submissions` table
- Add Server Actions for form handling
- Configure email notifications

### Phase 3: CMS Integration
- Consider Sanity or Contentful for:
  - Blog posts
  - Project gallery
  - Team members
  - Testimonials

### Phase 4: Analytics
- Add Google Analytics 4
- Set up conversion tracking for form submissions
- Track page engagement

## Code Quality

### Imports
Use path aliases: `@/components/ui`, `@/lib/utils`

### Types
TypeScript strict mode enabled. Define interfaces for:
- Props
- API responses
- Form data

### SEO
Every page has:
- Unique `metadata` export
- Proper heading hierarchy (single H1)
- Image alt text
- Semantic HTML

---
> Source: [agentlybot/odio-custom-builders](https://github.com/agentlybot/odio-custom-builders) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
