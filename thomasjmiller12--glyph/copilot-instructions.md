## glyph

> **Before ANY commit or push**, ensure:

# Glyph — Multiplayer Word Duel

## CRITICAL: Git Identity

**Before ANY commit or push**, ensure:
- **Author email**: `thomasjmiller12@gmail.com` (NEVER `thomas@billables.ai` or any work email)
- **Author name**: `Thomas Miller`
- **GitHub account**: `thomasjmiller12` (NEVER `TMiller0112`)

```bash
# Verify before committing:
git config user.email  # Must be thomasjmiller12@gmail.com
gh auth status         # Active account must be thomasjmiller12
```

If the email is wrong, fix it BEFORE committing:
```bash
git config user.email "thomasjmiller12@gmail.com"
git config user.name "Thomas Miller"
```

This is a personal project. Work account commits leak activity to the employer.

## Tech Stack
- Next.js 15 (App Router) + React 19
- Convex (database, real-time, auth, scheduled functions)
- Tailwind CSS 4
- Framer Motion for animations
- pnpm

## Development
- `pnpm dev` runs both Convex and Next.js
- Convex functions are in `convex/` — mutations, queries, scheduled functions
- Schema changes require `npx convex dev` to regenerate types

## Conventions
- Push to main, Vercel auto-deploys
- Dark-mode-first design, warm gold accent (#D4A574)
- Anonymous sessions via localStorage UUID, optional Convex Auth
- All word validation and feedback calculation happens server-side in Convex
- Never expose secretWord to the client
- Mobile-first responsive design (min tap target 44px)

## Key Files
- `convex/schema.ts` — all table definitions
- `convex/games.ts` — core game logic
- `convex/duels.ts` — duel mode logic
- `src/lib/words.json` — 5-letter dictionary
- `src/lib/feedback.ts` — client-side feedback display helpers

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thomasjmiller12)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/thomasjmiller12)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
