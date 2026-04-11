## kreeate

> Kreeate helps users turn plain-English bug reports and feature requests into concise GitHub issues, then submit them directly to selected repositories.

# AGENTS

## Project Purpose

Kreeate helps users turn plain-English bug reports and feature requests into concise GitHub issues, then submit them directly to selected repositories.

## Stack

- Next.js App Router + React + TypeScript
- NextAuth (GitHub OAuth)
- Drizzle ORM + PostgreSQL
- Tailwind CSS + shadcn/ui
- OpenRouter (via OpenAI SDK-compatible client)

## Working Conventions

- Keep behavior backward-compatible unless change is intentional and documented.
- Keep files under 500 lines (enforced by ESLint).
- Prefer reusable components for large UI sections.
- Keep API contracts explicit and validated with Zod.
- Use concise commit messages with clear intent.

## Local Commands

- `npm run dev` - run development server
- `npm run lint` - run ESLint
- `npm run build` - production build check
- `npx drizzle-kit migrate` - apply DB migrations

## Areas To Know

- `app/page.tsx` orchestrates homepage state and actions.
- `components/home/*` contains homepage presentation components.
- `app/api/generate/route.ts` handles prompt, validation, and model fallback.
- `app/api/submit/route.ts` handles GitHub issue submission.
- `app/api/preferences/route.ts` stores last/pinned repositories.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Logan1x)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Logan1x)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
