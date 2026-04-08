## me

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Nextra documentation site using the `nextra-theme-docs` theme. Nextra is a Next.js-based static site generator for documentation.

**Live site**: https://me-kappa-six.vercel.app/

Commits to `main` automatically deploy to Vercel.

## Commands

```bash
pnpm i          # Install dependencies
pnpm dev        # Start development server (localhost:3000)
pnpm build      # Build for production
pnpm start      # Start production server
```

## Architecture

- **Content**: Documentation pages live in `pages/` as `.mdx` files (Markdown with JSX support)
- **Navigation**: Controlled by `_meta.json` files in each directory - defines page order, titles, and external links
- **Theme config**: `theme.config.tsx` contains site-wide settings (logo, links, footer)
- **Next.js config**: `next.config.js` wraps Next.js with Nextra using `nextra-theme-docs`
- **Components**: React components in `components/` can be imported directly into MDX files

## Adding Pages

Create `.mdx` files in `pages/` and add entries to `pages/_meta.json` to control navigation order and titles.

## Skills

Available slash commands for Claude Code:

| Command | Purpose |
|---------|---------|
| `/nextra-dev` | Nextra-specific patterns, MDX pages, navigation, theme config |
| `/docs-engineer` | Documentation writing, information architecture, content patterns |
| `/commit` | Git commit workflow |
| `/code-review` | Code review |
| `/security-review` | Security review |
| `/apply` | Generate tailored job application (merged cover letter + resume DOCX) |

## Active Project: Job Application Generator

### What we're building
An app on this Nextra site that generates tailored job application materials. The flow:
1. Jayr finds a job posting (Indeed, etc.)
2. Pastes the job posting into the app
3. App generates: personalized intro/cover letter, customized CV, and interview prep questions
4. Each application gets a unique private URL (e.g. `/apply/j7kx9m2p`) so companies can't see each other's pages

### Key files
- **`.local/master-profile.md`** — Jayr's master CV/profile (human-readable, source of truth). GITIGNORED.
- **`.local/master-profile.json`** — Machine-readable version (needs syncing with .md). GITIGNORED.
- **`.gitignore`** — blocks `.local/` and `*.pdf` from being pushed

### Pending items (next session)
- [ ] Jayr to provide more context on each role in master-profile.md
- [ ] Confirm IBC Tanks dates (check T4/pay records)
- [ ] Confirm Trimorph start date (Jul 2010, Jun 2011, or Jul 2011?)
- [ ] Confirm Convergys dates (Nov 2009 or Nov 2010?)
- [ ] Add cleaning job details (company, dates, duties)
- [ ] Sync master-profile.json with the updated .md
- [ ] Start building the `/apply/[slug]` dynamic route and generation workflow

### Architecture decisions
- Nextra stays for the public portfolio. Private application pages are added alongside via dynamic Next.js routes.
- CV data stays local (`.local/`), only tailored/curated output gets pushed and deployed.
- Unique random slugs for each application URL (unguessable, no index page).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jarutosurano)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jarutosurano)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
