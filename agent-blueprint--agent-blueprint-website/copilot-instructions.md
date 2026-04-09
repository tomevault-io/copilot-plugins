## agent-blueprint-website

> Marketing site for [agentblueprint.ai](https://agentblueprint.ai).

# Agent Blueprint Website

Marketing site for [agentblueprint.ai](https://agentblueprint.ai).

## Tech Stack

- **Framework**: Next.js 16 (App Router)
- **Styling**: Tailwind CSS 4 + shadcn/ui components
- **Language**: TypeScript
- **Deployment**: Vercel (auto-deploys on push to `main`)

## Project Structure

```
src/
  app/           # Next.js App Router pages and API routes
  components/    # React components (ui/ for shadcn, sections/ for page sections)
  lib/           # Utilities and helpers
```

## Development

```bash
npm install      # Install dependencies
npm run dev      # Start dev server (localhost:3000)
npm run build    # Production build
npm run lint     # Run ESLint
```

## Git Workflow

This repo uses a **branch → preview → main** workflow. Vercel auto-deploys:
- **Push to `main`** → production deploy (live at agentblueprint.ai)
- **Push to any other branch** → preview deploy (unique `.vercel.app` URL)

### How to make changes

1. Create a branch for your work:
   ```bash
   git checkout -b my-changes
   ```
2. Make changes, commit early and often with clear messages
3. Run `npm run build` to verify nothing is broken
4. Push the branch:
   ```bash
   git push -u origin my-changes
   ```
5. Vercel will auto-generate a preview URL. Find it by running:
   ```bash
   gh pr list   # or check the Vercel dashboard
   ```
   The preview URL will look like: `agent-blueprint-website-<hash>.vercel.app`
6. Review the preview to confirm everything looks good
7. When ready to go live, merge to main and push:
   ```bash
   git checkout main
   git merge my-changes
   git push origin main
   ```

### Rules

- Never push directly to `main` without previewing first
- Always run `npm run build` before pushing
- Commit early and often with descriptive messages
- If a build fails on Vercel, the previous working version stays live (no downtime)

## Environment Variables

Environment variables are configured in Vercel (Project Settings > Environment Variables). Do not commit `.env` files.

- `SENDGRID_API_KEY` - Used by the contact form API route

## Team

- **Chris Faulkner** — technical cofounder, built all current code
- **Amy Glencross** — non-technical cofounder, has repo write access
  - If Amy is the one chatting: use plain language, explain technical terms, confirm before running anything destructive, and offer to handle git/CLI commands for her
  - See `SETUP-AMY.md` for her onboarding guide

## Workstream Tracking

**Read `WORKSTREAM.md` at the start of every session.** It has the current branch, what's been done, what's next, and open decisions.

**Update `WORKSTREAM.md` when work is done:**
- After committing/pushing, update the "What Was Done" section and check off completed items in "What Needs Doing Next"
- Add new items to "What Needs Doing Next" as they come up
- Keep "Active Branch" and "Working tree" status current
- If a session ends without pushing, note uncommitted work

## Key Conventions

- Components use shadcn/ui patterns (see `src/components/ui/`)
- Use Tailwind utility classes for styling, not CSS files
- Follow existing file and naming patterns in the codebase

## GEO/AEO Checklist

Every new page must follow these rules (see `docs/GEO-AEO-GUIDELINES.md` for templates and details):

1. **One H1 per page**, proper heading hierarchy (H1 > H2 > H3, no skipping)
2. **Add BreadcrumbList + WebPage JSON-LD schemas** using `breadcrumbSchema()` and `webPageSchema()` from `src/lib/metadata.ts`
3. **Direct-answer-first content structure** — lead with a clear answer, then expand with detail
4. **Update `src/app/sitemap.ts`** — add the new page with a real `modified` date (never `new Date()`)
5. **Update `public/llms.txt` and `/llms-full.txt`** with new page content
6. **Never block AI crawlers** in `robots.txt` — all listed bots must have `Allow: /`
7. **Keep `dateModified` in sync** — update WebPage schema and sitemap dates when page content changes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/agent-blueprint)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/agent-blueprint)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
