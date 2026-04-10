## reachify

> Reachify is a SaaS tool built for job seekers to find hiring manager emails, generate AI-personalized outreach emails, and send them in bulk via their own Gmail/Outlook account. This is NOT a sales tool — the UX, copy, and AI prompts are all designed for job seekers.

# Reachify — Claude Code Project Instructions

## Project Overview
Reachify is a SaaS tool built for job seekers to find hiring manager emails, generate AI-personalized outreach emails, and send them in bulk via their own Gmail/Outlook account. This is NOT a sales tool — the UX, copy, and AI prompts are all designed for job seekers.

## Tech Stack
- **Frontend/Backend:** Next.js 14 (App Router)
- **Styling:** Tailwind CSS + shadcn/ui
- **Database:** Supabase (PostgreSQL)
- **Auth:** Clerk (Google OAuth — doubles as Gmail send permission)
- **Email Finding:** Hunter.io API + Proxycurl API
- **Email Verification:** ZeroBounce API
- **AI Generation:** Anthropic Claude API (claude-sonnet-4-20250514)
- **Email Sending:** Gmail API via Google OAuth
- **Payments:** Stripe (subscriptions + credit top-ups)
- **Hosting:** Vercel

## Project Structure
```
reachify/
├── app/
│   ├── (auth)/
│   │   ├── sign-in/
│   │   └── sign-up/
│   ├── (dashboard)/
│   │   ├── layout.tsx
│   │   ├── dashboard/
│   │   │   └── page.tsx
│   │   ├── finder/
│   │   │   └── page.tsx
│   │   ├── campaigns/
│   │   │   ├── page.tsx
│   │   │   └── [id]/
│   │   │       └── page.tsx
│   │   ├── compose/
│   │   │   └── page.tsx
│   │   └── settings/
│   │       └── page.tsx
│   ├── api/
│   │   ├── emails/
│   │   │   ├── find/route.ts
│   │   │   ├── verify/route.ts
│   │   │   └── enrich/route.ts
│   │   ├── ai/
│   │   │   └── generate/route.ts
│   │   ├── send/
│   │   │   └── route.ts
│   │   ├── campaigns/
│   │   │   └── route.ts
│   │   ├── credits/
│   │   │   └── route.ts
│   │   └── webhooks/
│   │       └── stripe/route.ts
│   ├── layout.tsx
│   └── page.tsx (landing page)
├── components/
│   ├── ui/ (shadcn components)
│   ├── dashboard/
│   ├── finder/
│   ├── compose/
│   └── campaigns/
├── lib/
│   ├── supabase.ts
│   ├── hunter.ts
│   ├── proxycurl.ts
│   ├── zerobounce.ts
│   ├── gmail.ts
│   ├── anthropic.ts
│   ├── stripe.ts
│   └── utils.ts
├── hooks/
├── types/
│   └── index.ts
├── middleware.ts
├── .env.local
└── PRD.md
```

## Core Workflows (Build in This Order)

### Phase 1 — Foundation
1. Set up Next.js 14 project with Tailwind + shadcn/ui
2. Configure Clerk auth with Google OAuth (request Gmail send scope)
3. Set up Supabase with full schema (see DATABASE.md)
4. Create dashboard layout with sidebar navigation
5. Implement credit tracking system

### Phase 2 — Email Finding
1. Hunter.io domain search (company name → list of emails)
2. Proxycurl LinkedIn enrichment (LinkedIn URL → email)
3. ZeroBounce email verification
4. Email results table with select/deselect UI

### Phase 3 — AI Generation
1. User profile setup (resume text, background, target role)
2. Claude API integration for personalized email generation
3. Per-recipient unique email generation
4. Email preview + edit before send

### Phase 4 — Bulk Send
1. Gmail API OAuth send flow
2. Send throttling (max 20/day, 5-min delay between sends)
3. Campaign creation and tracking
4. Open/reply tracking via pixel + link wrapping

### Phase 5 — Monetization
1. Stripe subscription setup (Free, Job Seeker $19, Premium $39)
2. Credit system for lookups and sends
3. Credit top-up purchases
4. Usage dashboard

## Key Business Rules
- Free plan: 10 email lookups/month, 5 AI emails/month, no bulk send
- Job Seeker ($19/month): 100 lookups, 50 AI sends, bulk send up to 20/day
- Premium ($39/month): 300 lookups, 200 AI sends, bulk send up to 50/day
- Credit top-ups: $5 for 50 extra lookups, $3 for 20 extra AI sends
- NEVER send from Reachify's own email domain — always send from user's Gmail
- Throttle bulk sends to avoid spam flags (max 20/day, randomized delay 3–8 min between sends)
- Always verify emails before allowing send to reduce bounce rate
- Hard cap API usage per user per billing period — enforce server-side

## AI Email Generation Rules
When calling Claude API to generate emails:
- Always generate ONE unique email per recipient — never duplicate
- Personalize using: recipient's name, company, role, any LinkedIn summary available
- Write in job-seeker voice, NOT sales voice
- Emails should be 100–150 words max — short is better for cold outreach
- Include subject line + body
- Never sound desperate or generic
- Tone options: Professional, Conversational, Bold
- Always include a clear ask (15-minute call, referral to open role, etc.)

## Error Handling Standards
- All API routes must return consistent error shapes: `{ error: string, code: string }`
- All external API calls must be wrapped in try/catch with fallback error messages
- Credit deduction must happen AFTER successful API response, never before
- If email send fails mid-campaign, log failure and continue with remaining recipients
- Show user a detailed send report after each campaign

## Security Rules
- All API routes must verify Clerk session server-side before processing
- Never expose Hunter.io, Proxycurl, or ZeroBounce API keys to the client
- Stripe webhooks must verify signature before processing
- Rate limit all API routes (use Vercel Edge middleware)
- Sanitize all user inputs before passing to AI prompts

## UI/UX Principles
- Clean, minimal design — users are stressed job seekers, not enterprise sales reps
- Every action should take < 3 clicks
- Always show credit balance in header
- Show cost in credits before user confirms an action
- Loading states on every async action
- Mobile responsive — job seekers are often on mobile

## Environment Variables Required
See ENVIRONMENT.md for full list. Never hardcode any API keys.

## Database
See DATABASE.md for complete Supabase schema and RLS policies.

## API Integrations
See API_INTEGRATIONS.md for complete integration details, rate limits, and error handling for each external API.

## Coding Conventions
- Use TypeScript strictly — no `any` types
- Use server actions or API routes for all external API calls — never call from client
- Use Supabase client for DB — never raw SQL from API routes
- Use `zod` for all input validation
- Components: functional, named exports
- Use `cn()` utility for conditional classnames
- Prefer server components, use client components only when needed (interactivity)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aarshrpatel)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/aarshrpatel)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
