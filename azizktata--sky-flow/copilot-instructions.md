## sky-flow

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Sky-Flow is a Next.js 15 corporate website for an import/export logistics company. Built with TypeScript, React 19, and App Router pattern.

## Commands

```bash
npm run dev      # Start dev server (Turbopack enabled)
npm run build    # Production build
npm run start    # Run production server
npm run lint     # ESLint (next/core-web-vitals + next/typescript)
```

Package manager: pnpm (v9.15.0)

## Architecture

### Data Flow
```
src/app/page.tsx (Server Component)
    ↓
src/utils/fetchDataFromWP.ts
    ↓
WordPress REST API (ACF fields)
    ↓
Client components render data
```

### Key Patterns

- **Server Components**: `page.tsx` fetches data server-side
- **Client Components**: Marked with `"use client"` for interactivity (header, contactForm, footer, homeCarousel)
- **Server Actions**: Email sending via `src/utils/email.js` using Nodemailer
- **Form Handling**: React Hook Form + Zod validation in contactForm.tsx

### UI Stack
- **shadcn/ui** (new-york style): button, input, label, form, carousel
- **NextUI**: navbar, card, image
- **Styling**: Tailwind CSS with class-variance-authority (CVA) for variants

### Path Alias
`@/*` maps to `./src/*`

## Environment Variables

```
BASE_URL=           # WordPress API URL
EMAIL_USER=         # OVH SMTP email
EMAIL_PASS=         # OVH SMTP password
NEXT_PUBLIC_EMAIL_TO=  # Contact form recipient
HOSTNAME=           # For Next.js image optimization
```

## Brand Colors

- Primary Blue: `#02446E`
- Secondary Purple: `#C570B8`
- Dark Navy: `#0E305D`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/azizktata)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/azizktata)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
