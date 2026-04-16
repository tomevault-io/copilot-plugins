## pasuhealth

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PASU Health is a Next.js 15 application for a workplace mental health and wellbeing provider. The site offers:

Mental health training courses (virtual, face-to-face, self-guided)
Workplace wellness consultancy services
Mental health risk assessment tool
Future wellness platform (pasu.io)
Built with Next.js App Router, TypeScript, Tailwind CSS v4, Sanity CMS, and Resend for email delivery.

## Commands

### Development

- `npm run dev` - Start development server with Turbopack
- `npm run build` - Build for production
- `npm start` - Start production server
- `npm run lint` - Run ESLint

### Environment Variables

Required environment variables (in `.env.local`):

- `NEXT_PUBLIC_SANITY_PROJECT_ID` - Sanity project ID
- `NEXT_PUBLIC_SANITY_DATASET` - Sanity dataset name
- `RESEND_API_KEY` - Resend API key for email functionality

## Architecture

### Multi-Domain Setup

The application supports multiple domains through middleware:

- Main site: `pasuhealth.com`
- Admin/CMS access: `admin.pasuhealth.com` (rewrites to `/admin` route)
- Middleware in `middleware.ts` handles domain-based routing

### Content Management (Sanity CMS)

- Sanity configuration in `sanity/` directory
- Client setup: `sanity/lib/client.ts`
- Data access functions: `sanity/accessData/` (e.g., `getTrainingCoursesData.ts`)
- Training course content is managed entirely through Sanity CMS
- Sanity Vision available at admin subdomain for content querying/debugging

### Key Application Features

**Mental Health Risk Checker** (`app/mental-health-risk-checker/`)

- Multi-step assessment tool with location-specific questions (England/Wales/Scotland/NI)
- Two assessment sections: Legal/Policy compliance (Section A) and Psychosocial factors (Section B)
- PDF report generation using `@react-pdf/renderer`
- Complex state management with progress tracking
- Types defined in `types/riskChecker.ts`
- Utility functions in `riskCheckerUtils.ts`
- Email submission via server action `actions/assessmentEmail.ts`

**Form System**
Three main form types with parallel implementations:

1. Contact forms (`components/contactForm/`)
2. Callback request forms (`components/requestCallbackForm/`)
3. Training enquiry forms (`components/trainingEnquiryForm/`)

Each form follows pattern:

- Form component with react-hook-form + Zod validation
- Server actions in `app/actions/` for email sending via Resend
- Separate FormStatus and SubmitButton components
- Validation schemas defined with Zod in server actions

### Server Actions Pattern

All server actions in `app/actions/`:

- `assessmentEmail.ts` - Mental health assessment results
- `contact.ts` - General contact form
- `enquiry.ts` - Training enquiry submissions
- `requestCallback.ts` - Callback request handling
- All use Resend for email delivery
- All marked with "use server" directive

### Component organisation

- Page-specific components in `components/` (e.g., `HeroSection.tsx`, `OurCourses.tsx`)
- Form components grouped in subdirectories
- Container components in `components/containers/`
- Shared components like `Nav.tsx`, `VideoFooter.tsx` in root of `components/`

### Styling

- Tailwind CSS v4 with PostCSS
- Global styles in `app/globals.css`
- Red Hat Display font loaded via next/font/google
- Custom selection color: emerald-700/25

### PDF Generation

- API route: `app/api/render-to-pdf/route.ts`
- PDF components built with `@react-pdf/renderer`
- Generator utility: `lib/pdf-generator.tsx`
- Download button component: `components/PDFDownloadButton.tsx`

### Analytics

Google Tag Manager integrated via `@next/third-parties/google` in root layout (GTM-5TRMLKZQ)

### Image Handling

Remote image patterns configured for:

- Sanity CDN (`cdn.sanity.io`)
- Unsplash (`images.unsplash.com`)

## TypeScript Configuration

- Strict mode enabled
- Path aliases: `@/` maps to root, `@components/`, `@actions/`, `@public/`
- Module resolution: bundler

## Code Patterns

### Data Fetching

Use Sanity client functions from `sanity/accessData/` rather than inline queries. Follow the pattern in `getTrainingCoursesData.ts` for consistent GROQ queries.

### Form Handling

1. Define Zod schema in server action file
2. Validate form data server-side
3. Use react-hook-form for client-side validation
4. Return structured response with success/error states
5. Display FormStatus component based on submission state

### Server Components by Default

The app uses React Server Components. Mark client components with "use client" directive when needed (forms, interactive components).

### Risk Checker State Management

The RiskChecker component uses complex state management with multiple assessment steps. When modifying:

- Maintain step flow: introduction → location → sectionA → sectionB → contact → results
- Update progress calculation in `calculateProgress()` utility
- Ensure all question types have corresponding risk calculations
- Preserve PDF generation compatibility

## Don't make these mistakes!

### Forgetting to escape apostrophes

when writing JSX always use '&apos;' instead of ' to avoid rendering issues in production.

### Creating duplicate code when existing implementations are available

Before creating new components, actions, or utilities, ALWAYS search for existing implementations first:

- Use Grep or Glob to search for similar components, functions, or patterns
- Check `components/` for reusable UI components
- Check `app/actions/` and `app/*/actions.ts` for existing server actions
- Look for similar functionality in related features (e.g., if adding admin features, check partner features for reusable patterns)
- Reuse and adapt existing code rather than duplicating functionality
- This maintains consistency, reduces code duplication, and ensures patterns are followed across the codebase

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thomassleeman) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
