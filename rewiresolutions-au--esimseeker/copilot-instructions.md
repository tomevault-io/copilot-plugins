## esimseeker

> You are an expert Next.js (App Router), React, Tailwind CSS, and TypeScript developer building the eSIMSeeker website.

# eSIMSeeker Cursor Rules

You are an expert Next.js (App Router), React, Tailwind CSS, and TypeScript developer building the eSIMSeeker website.

## Project Context
eSIMSeeker is a minimalist, AI-powered eSIM comparison platform. Users chat with an AI Wizard to find the best data plan for their trip. The site is highly optimised for SEO and performance, generating revenue through affiliate links (e.g., Holafly, Airalo).

## Core Principles
1. **Minimalism:** No unnecessary UI elements. Focus entirely on conversion and trust.
2. **Performance:** Use Server Components by default. Keep client-side JS to an absolute minimum.
3. **Type Safety:** Strict TypeScript everywhere. No `any` types.

## Tech Stack
- Framework: Next.js 14+ (App Router)
- Language: TypeScript
- Styling: Tailwind CSS + Shadcn UI
- AI: Vercel AI SDK (v3/v4) + OpenAI GPT-4o
- Icons: Lucide React

## File Structure & Routing
- `app/page.tsx` - Homepage
- `app/wizard/page.tsx` - AI Chat Interface
- `app/destinations/page.tsx` - SEO Hub
- `app/esim/[country]/page.tsx` - Dynamic pSEO country pages
- `components/` - Shared UI components (see `DESIGN.md` for full list)
- `lib/` - Utility functions, API clients, and AI tool definitions

## Reference Documents
Before writing any code, you MUST reference the provided design and spec documents:
- `@DESIGN.md` - Full site architecture, component list, and brand tokens.
- `@CHAT_SPEC.md` - Detailed logic and UI states for the AI Wizard.

## Coding Standards
- **Components:** Use functional components with `const Component = () => {}` syntax.
- **Tailwind:** Use the custom brand tokens defined in `DESIGN.md` (e.g., `text-brand-navy`, `bg-brand-red`).
- **AI Integration:** Implement tool calling for plan retrieval. Render plan cards inline within the chat feed using the AI SDK's `render` or `components` props.
- **SEO:** Every page must export a `generateMetadata` function. Country pages must include JSON-LD structured data.

When asked to build a specific page or component, always ask the user to provide the corresponding wireframe image from the `wireframes/` folder for visual reference.

---
> Source: [rewiresolutions-au/esimseeker](https://github.com/rewiresolutions-au/esimseeker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
