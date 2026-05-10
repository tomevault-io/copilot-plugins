## timeline

> Build and maintain a personal portfolio website with Astro + Tailwind that resembles modern Twitter's information architecture:

# AGENTS Instructions

## Project Goal
Build and maintain a personal portfolio website with Astro + Tailwind that resembles modern Twitter's information architecture:
- Left pane: identity + navigation
- Center pane: content timeline
- Right pane: dynamic context (Now + GitHub activity)

## Core Product Rules
- Preserve the three-pane desktop experience on large screens.
- Preserve mobile responsiveness with a bottom nav and single main feed pane.
- Keep visual style close to modern Twitter, but do not copy Twitter branding assets.
- Keep content sourced from real resume/profile data unless user asks otherwise.
- Keep right-pane widgets lightweight and dynamic.

## Technical Rules
- Use Astro with file-based routes.
- Use TailwindCSS for styling.
- Keep components small and reusable.
- Put content in `src/data/site.ts` and render via components/layouts.
- Validate every major change with `npm run build`.

## Git Rules
- Commit after each major slice of work.
- Commit messages must be simple one-sentence statements.
- Do not use commit prefixes like `feat:`, `fix:`, `chore:`, `docs:`, or similar.

## Deployment Rules
- Target platform: Cloudflare Pages.
- Production domain: `kakaruto.com`.
- Ensure builds output static assets to `dist/`.

## Collaboration Rules
- If details are missing, make a reasonable assumption and continue.
- Stop only for hard blockers that require external credentials, permissions, or user decisions.

---
> Source: [kaka-ruto/timeline](https://github.com/kaka-ruto/timeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
