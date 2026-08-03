## portfolio

> Personal developer portfolio for Zeeshan Mehmood, a junior full-stack developer

# Zeeshan Mehmood — Portfolio (zeeshanmehmood.be)

## Project Overview
Personal developer portfolio for Zeeshan Mehmood, a junior full-stack developer
based in Geel, Belgium, graduating from Thomas More University in June 2026.
Built with Next.js 15 (App Router), TypeScript, Tailwind CSS, shadcn/ui, and
a Vercel-hosted AI chatbot powered by the Anthropic Claude API.

All personal content (bio, projects, skills, experience) is in @CONTENT.md.

## Commands
```bash
npm run dev        # Start dev server on http://localhost:3000
npm run build      # Production build
npm run lint       # ESLint check
npm run typecheck  # TypeScript check (tsc --noEmit)
```
Run typecheck and lint after every major change. Fix all errors before committing.

## Tech Stack
- **Framework:** Next.js 15 App Router with TypeScript
- **Styling:** Tailwind CSS + shadcn/ui components
- **Animations:** Framer Motion
- **Icons:** Lucide React
- **Chatbot:** Vercel AI SDK (`ai` + `@ai-sdk/anthropic`) with streaming
- **Rate limiting:** Upstash Redis (`@upstash/redis` + `@upstash/ratelimit`)
- **Analytics:** Vercel Analytics (`@vercel/analytics`)
- **Fonts:** Geist (already in Next.js 15)

## Architecture
```
app/
  layout.tsx           # Root layout with metadata, fonts, analytics
  page.tsx             # Home page — assembles all sections
  api/
    chat/
      route.ts         # POST — streams chatbot responses via Claude API
components/
  sections/            # Hero, About, Experience, Projects, Skills, Contact
  ui/                  # shadcn/ui components + custom shared components
  chatbot/             # ChatButton, ChatWindow, ChatMessage
lib/
  content.ts           # Re-exports all personal data from CONTENT.md as TS
  ratelimit.ts         # Upstash rate limiter config
```

## Design System
- **Theme:** Dark. Background `slate-950`, cards `slate-900`, borders `slate-800`
- **Accent:** Indigo-to-purple gradient (`from-indigo-500 to-purple-600`)
- **Text:** White headings, `slate-300` body, `slate-400` muted
- **Radius:** `rounded-xl` on cards, `rounded-full` on badges
- **Motion:** Subtle fade-up on scroll (Framer Motion `viewport` prop)
- **Typography:** Geist Sans. Headings bold, tracking-tight
- Make it look like a senior dev's portfolio, not a student template

## Pages / Sections (in order on the homepage)
1. **Navbar** — sticky, frosted glass, name left + nav links + "Hire Me" CTA button
2. **Hero** — name, role with typing animation, location, two CTA buttons, subtle grid background
3. **About** — short bio, photo placeholder, personality note (tourism + books)
4. **Skills** — grouped by category (Frameworks, Languages, Cloud, Tools) as icon badges
5. **Experience** — vertical timeline, most recent first
6. **Projects** — card grid, each with title, description, tech stack badges, GitHub + Live links
7. **Achievements** — highlight card for the LegoLib "Best Project" award
8. **Education** — two entries, timeline style
9. **Contact** — email link + LinkedIn link + contact form (Formspree or mailto fallback)
10. **Chatbot** — floating button bottom-right, opens a chat drawer, streams responses

## Chatbot Rules
- API route must NEVER expose the ANTHROPIC_API_KEY to the client
- Implement rate limiting: max 20 messages per IP per hour
- Use prompt caching for the system prompt (CV content is static)
- Stream responses using Vercel AI SDK `streamText`
- Include 3 suggested starter questions under the input
- The bot talks ABOUT Zeeshan, never AS Zeeshan

## Environment Variables (create .env.local)
```
ANTHROPIC_API_KEY=           # From console.anthropic.com
UPSTASH_REDIS_REST_URL=      # From upstash.com (free tier)
UPSTASH_REDIS_REST_TOKEN=    # From upstash.com (free tier)
NEXT_PUBLIC_FORMSPREE_ID=    # From formspree.io (free tier, for contact form)
```

## Existing Portfolio to Reference
Visit https://zeeshanmehmood59640.github.io/Profile/ to check the current
design and copy any content not already in CONTENT.md. The site is a SPA
(React + hash routing). Use curl or browser fetch to read the JS bundle if
the HTML shell is empty.

## Coding Rules
- TypeScript strict mode — no `any` types
- All components use named exports
- `use client` only when you need interactivity or browser APIs
- Image optimization: use `next/image` for all images
- Responsive: mobile-first, test at 390px and 1280px breakpoints
- Accessibility: all images need `alt` text, buttons need `aria-label`
- SEO: page has complete metadata in `app/layout.tsx`
- Commit after each major section is complete and working

## IMPORTANT
When starting a new session, read CONTENT.md first before writing any code.
Do not invent projects, skills, or achievements not listed in CONTENT.md.

---
> Source: [zeeshanmehmood59640/portfolio](https://github.com/zeeshanmehmood59640/portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
