## ai-travel-agent

> A mobile-first travel planning app built with React, TypeScript, Vite, and TailwindCSS.

# CLAUDE.md — AI Travel Agent

## Project Overview
A mobile-first travel planning app built with React, TypeScript, Vite, and TailwindCSS.
Users enter travel details and receive trip recommendations powered by the OpenAI API
via a Cloudflare Worker backend.

Live: https://hawiz-ai-travel-agent.netlify.app/
Backend: https://github.com/HawiCaesar/ai-travel-agent-worker

## How I Use AI in This Project
I write requirements and design decisions in plain language (see INIT.md, STRETCH_GOALS.md),
then use Cursor/Claude to generate detailed execution plans (plan.md, stretch.md) which I
review before building. Design tokens and component conventions are documented and used
as context for AI-generated code (see COLOR_SCHEME.md).

See E2E_TEST_PLAN.md for complete automated testing strategy, setup instructions, and all test scenarios.

I review all AI-generated code before accepting it — particularly checking that styling
matches the design tokens, TypeScript types are explicit, and accessibility attributes
are present.

## Tech Stack
- React 19 with TypeScript
- Vite
- TailwindCSS with custom design tokens
- React Router DOM v7
- React Context API for state management
- OpenAI API (via Cloudflare Worker)

## Design Tokens
Defined in tailwind.config.js. Always use these — never hardcode hex values.

| Token               | Value     | Usage                        |
|---------------------|-----------|------------------------------|
| bg-brand-bg         | #F2FFFF   | Page background (all pages)  |
| bg-brand-button     | #4BDCB0   | Primary button background    |
| border-brand-border | #000000   | All borders (4px solid)      |
| bg-brand-card       | #BBF7F7   | Results page cards           |

## Component Conventions
- One component per file, PascalCase filenames: `TravelFormPage.tsx`, `Button.tsx`
- Hooks: camelCase prefixed with "use" — `useFormValidation.ts`
- Event handlers prefixed with "handle" — `handleSubmit`, `handleGoBack`
- Use early returns for cleaner conditional rendering
- No class components — functional components with hooks only

## Styling Rules
- Use design tokens above — no arbitrary Tailwind values like `bg-[#4BDCB0]`
- No inline styles
- Mobile-first: base styles target mobile, use `sm:` and `md:` for larger screens
- Buttons: 4px solid black border, brand-button background, brand body font
- Cards: bg-brand-card, `shadow-[0_4px_8px_rgba(0,0,0,0.25)]`, no border, rounded-3xl

## Typography
- Font family: Inter (loaded from Google Fonts)
- `custom-ai-travel-agent-font-body` — buttons and form inputs (Inter 700/bold, 100% line-height)
- `custom-ai-travel-agent-font-cards` — results page cards only (Inter 400/regular, 100% line-height)

## Accessibility Requirements
- All inputs must have associated labels or aria-label
- Buttons must have descriptive text or aria-label
- Form validation errors must be associated with their input via aria-describedby
- Keyboard navigation must work across all interactive elements
- Minimum touch target size: 44x44px

## State Management
- Form data lives in TravelContext (src/context/TravelContext.tsx)
- Local UI state (loading, validation errors) stays in the component via useState
- Do not put UI state into context

## File Structure
```
src/
  components/   # Reusable UI components (Button.tsx)
  pages/        # LandingPage, TravelFormPage, ResultsPage
  context/      # TravelContext
  types/        # TypeScript interfaces (index.ts)
  utils/        # dateFormatters.ts and other helpers
```

## Key Reference Files
- INIT.md — original requirements I wrote
- STRETCH_GOALS.md — feature additions I defined
- plan.md — AI-generated execution plan from INIT.md
- stretch.md — AI-generated execution plan from STRETCH_GOALS.md
- COLOR_SCHEME.md — design token documentation
- E2E_TEST_PLAN.md — comprehensive e2e testing strategy

---
> Source: [HawiCaesar/ai-travel-agent](https://github.com/HawiCaesar/ai-travel-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
