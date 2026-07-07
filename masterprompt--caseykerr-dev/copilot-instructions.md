## caseykerr-dev

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

- `npm run dev` - Start development server (localhost:3000)
- `npm run build` - Production build (outputs to `out/` directory)
- `npm run lint` - Run ESLint
- `npm test` - Run the Vitest suite once (CI mode)
- `npm run test:watch` - Vitest in watch mode
- `npm start` - Start production server (not used for static export)

## Deployment

- **Live site:** https://caseykerr.dev
- **Repo:** https://github.com/masterprompt/caseykerr.dev
- Deploys to GitHub Pages via GitHub Actions on push to `main`
- Custom domain configured with DNS pointing to GitHub Pages

## Architecture

Personal portfolio site for Casey Kerr (Senior Full Stack Software Engineer).

**Stack:**
- Next.js 16 with App Router and static export
- TypeScript
- Tailwind CSS v4

**Key directories:**
- `src/app/` - App Router pages and layouts
- `public/` - Static assets (including `kerrsoft-logo.png` for favicon)
- `docs/` - Profile documentation (gitignored, contains personal info for site content)
- `out/` - Build output (generated, gitignored)

## Project Status

**Completed:**
- Migrated from CRA to Next.js with static export
- GitHub Pages deployment working
- Custom domain (caseykerr.dev) configured
- Profile document compiled from resume (`docs/profile.md`)

**Next steps:**
1. Design site structure/pages based on profile content
2. Build out the actual portfolio pages:
   - Hero/landing section
   - About/professional summary
   - Skills/tech stack visualization
   - Career highlights & accomplishments
   - Projects (consulting work, personal/game dev)
   - Contact/links
3. Styling and polish

## About Casey (for content reference)

- Senior Full Stack Engineer (React, Node.js, AWS)
- Founder of Kerrsoft LLC (consulting) and Waukesha Makerspace (501(c)(3) nonprofit)
- Background spans PLC/industrial automation → enterprise software → modern cloud-native
- Published 4 games, prototyped 30+ more
- Motto: "Complexity embedded in simplicity"

See `docs/profile.md` for full details on experience, skills, and accomplishments.

## Agent skills

### Issue tracker

Local markdown under `.scratch/<feature-slug>/`. See `docs/agents/issue-tracker.md`.

### Triage labels

Canonical defaults (`needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`), recorded as a `Status:` line in each issue file. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context — `CONTEXT.md` + `docs/adr/` at the repo root. See `docs/agents/domain.md`.

---
> Source: [masterprompt/caseykerr.dev](https://github.com/masterprompt/caseykerr.dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
