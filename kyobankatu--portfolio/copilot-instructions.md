## portfolio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Mandatory Rules

- Do NOT modify any files unless explicitly instructed.
- Do NOT refactor existing code unless clearly requested.
- Prefer minimal, localized changes over large improvements.
- Stability and existing behavior are more important than code cleanliness.

## Change Proposal Requirement

Before making any code changes:
- Explain what will be changed
- Explain why it is necessary
- Describe potential risks or side effects

Wait for explicit approval before proceeding.

## Security Rules

- Never request or output secrets, API keys, or credentials.
- Do not log or print personal data.
- Assume production-like constraints even in development.

## Cost Awareness

- Keep responses concise.
- Avoid repeating large code blocks unless necessary.
- Prefer explanation over full implementation when possible.

## Notation

- Indent must be 4
- Don't insert extra spaces
- Don't omit "{}"
- Don't use left symbol like ">" and ">=". Please use right symbol like "<" and "<="
- Avoid using goto
- Consider readability

## Model Usage Policy

- Use the Default (recommended) model for all tasks.
- The Default model is currently Sonnet.
- Do NOT switch to Opus unless explicitly instructed by the user.
- Prefer lower-cost models unless higher capability is required.

## Commands

```bash
npm run dev      # Start dev server with HMR
npm run build    # Production build → .next/
npm run start    # Serve production build locally
```

No test framework is configured.

## Architecture

Next.js 15 (App Router) + React 19 portfolio. The home page is statically generated at build time.

- **`app/layout.js`** — root layout; sets `<html lang="ja">` and imports `globals.css`
- **`app/page.js`** — home route; renders `<Home />`
- **`app/globals.css`** — global styles: dark navy gradient background (`#0b1220`), animated pseudo-element orbs on `body::before`/`body::after`, light-mode overrides via `prefers-color-scheme`
- **`components/Home.jsx`** — `'use client'` component containing the entire page: hero, profile card, skill badges, CTA buttons, placeholder Projects/Contact sections, and JS-driven orb animation
- **`components/Home.module.css`** — scoped styles for `Home.jsx`
- **`public/avatar-placeholder.png`** — avatar image served as `/avatar-placeholder.png`

## Styling conventions

- Dark theme by default; light mode via `@media (prefers-color-scheme: light)`
- Gold accent color: `#bfa046` / `#d7b262`
- Component styles use CSS Modules; global baseline is in `app/globals.css`
- Orb animation runs two layers: CSS pseudo-elements on `body` (static) and a `useEffect`/`useState`/`setInterval` driven layer in `Home.jsx` (dynamic, spawns and removes orbs by ID)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kyobankatu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
