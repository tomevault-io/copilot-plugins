## damonsharp-next

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # Start development server
npm run build    # Production build
npm run lint     # ESLint validation
npm run email    # React Email preview server (for email template dev)
```

No test suite is configured.

## Architecture

Personal portfolio site for Damon Sharp. Next.js 15 App Router with **WordPress as a headless CMS** accessed via GraphQL.

### Data Flow

All content (projects, resume, navigation, site settings) is fetched from a WordPress GraphQL API:
- Production: `https://dws.230studios.com/graphql` (via `NEXT_PUBLIC_WP_GRAPHQL_URL`)
- Local dev: `https://damonsharp-wp.test/graphql`

GraphQL queries live in `lib/queries/`. Apollo Client is initialized in `lib/apolloClient.js` and wrapped for client components via `components/ApolloWrapper.jsx`. Pages use server-side Apollo queries directly.

### Pages & Routing

| Route | Source |
|-------|--------|
| `/` | Home: hero, featured project banner, recommendations |
| `/projects` | Projects listing from WordPress |
| `/projects/[slug]` | Individual project detail |
| `/resume` | Full resume: experience, skills, recommendations |
| `/contact` | Contact form |

### Contact Form Flow

1. `components/ContactForm.jsx` — client form with Zod validation (`lib/validators.ts`)
2. `actions/sendEmail.js` — server action: validates, sends via Resend API
3. Email templates in `components/emails/` (React Email components)
4. Spam protection: math challenge field `juststop` (answer: "6" or "six")

### Styling

Tailwind CSS v4 (PostCSS). Custom theme defined in `assets/styles/global.css` using oklch color space:
- `--color-primary`: dark navy
- `--color-accent`: tan/beige
- `--color-neutral`: off-white
- `--color-secondary`: purple

shadcn/ui components are in `components/ui/` (style: "new-york", base color: neutral).

### Key Environment Variables

```
NEXT_PUBLIC_WP_GRAPHQL_URL   # WordPress GraphQL endpoint
RESEND_API_KEY               # Transactional email
RESEND_FROM / RESEND_TO      # Email addresses
NODE_TLS_REJECT_UNAUTHORIZED=0  # Dev only (self-signed cert for local WP)
```

### Tech Notes

- **No TypeScript** — JSX/JS throughout; Zod used for runtime validation only
- **React 19 / Next.js 15** — uses Server Components by default; client components marked with `"use client"`
- **Icons**: FontAwesome (brands/social) + Lucide React (UI icons)
- **Path alias**: `@/` maps to the repo root
- Image remote patterns configured in `next.config.mjs` for WordPress domains

---
> Source: [damonsharp/damonsharp-next](https://github.com/damonsharp/damonsharp-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
