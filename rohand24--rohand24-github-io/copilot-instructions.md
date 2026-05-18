## rohand24-github-io

> This repo should become a production-ready personal website. You are expected to edit code, run commands, and provide proof that the site builds and is deployable.

# AGENTS.md — Build + Deploy Rohan’s Personal Website

This repo should become a production-ready personal website. You are expected to edit code, run commands, and provide proof that the site builds and is deployable.

## Goal
Create a fast, clean, static personal site that is sourced from local files in this repo (no scraping of external sites) and deploy it publicly using GitHub Pages.

The site should highlight:
- Home: concise intro + key links
- About: longer bio
- Research/Publications: rendered from `publications.bib`. Organized in descending ones first.
- Projects: rendered from `projects.json`
- CV: a page plus a downloadable CV file placed under `public/`
- Contact: email + links

## Current site direction and implementation notes
The site was originally tuned around translational work for an award application, but the current direction is a job-application portfolio for both industry and academic roles. Preserve a professional biomedical AI / translational research tone, and avoid making it sound like a generic student project portfolio.

Current navigation labels intentionally differ from some route names to avoid breaking deployed links:
- Top nav label `Publications` points to `/research`, which renders publications from `publications.bib`.
- Top nav label `Research` points to `/projects`, which renders the research portfolio from `projects.json`.
- Keep these routes stable unless the user explicitly asks to rename URLs. If cleaner URLs are requested later, prefer adding compatibility pages/redirects rather than breaking existing links.

About page structure:
- `About My Research` should be expanded by default and remain collapsible.
- `Selected publications` should appear before `Translational track record`.

Analytics:
- Cloudflare Web Analytics is configured via `site.config.json` under `integrations.cloudflareWebAnalyticsToken`.
- The shared layout conditionally injects Cloudflare's beacon script when that token is set.
- Do not add additional analytics or raw-IP logging unless the user explicitly asks.

Visual direction:
- Current palette is slate/teal/blue: professional, medical/technical, and suitable for academic plus industry applications.
- If improving design, keep the core palette unless asked otherwise, but strengthen hierarchy and portfolio polish rather than overhauling the brand.

## Source-of-truth inputs
Use local files as the only content source.
- `resume.md` is the canonical resume text.
- Publications MUST be loaded from: `publications.bib` (do not invent citations).
- `projects.json` is the canonical projects list.
- `site.config.json` is canonical for links/metadata.

Do not scrape LinkedIn or Google Scholar. Only link out to them.

## Framework and structure
Default stack is Astro (static site). If Astro is not already initialized, initialize it.
Use these conventions:
- Pages: `src/pages/*`
- Components: `src/components/*`
- Data: `src/data/*` (preferred) OR repo root (choose one, be consistent)
- Static assets (CV, images): `public/*`

Put the downloadable CV in `public/cv/` and link it from `/cv`.
If only a DOCX exists, keep it downloadable; optionally also create a PDF if conversion is available without external services, otherwise leave as DOCX.

## Local commands to run (required)
You must run these commands and ensure they succeed before you finish:
1) `npm install`
2) `npm run build`

Also run at least one of the following to confirm the site serves:
- `npm run preview` (preferred)
- or document why preview cannot be run, but only after confirming build succeeds

If you add typechecking/linting, ensure they pass:
- `npm run lint` (optional)
- `npx astro check` (optional)

## Definition of Done (acceptance criteria)
The work is complete only when:
- Site builds successfully (`npm run build` passes).
- Publications page renders entries from `publications.bib`.
- Projects page renders from `projects.json`.
- CV page provides a working download link to a file under `public/cv/`.
- A GitHub Actions workflow exists that deploys to GitHub Pages.
- The Astro config supports GitHub Pages base path when required.

## GitHub Pages deployment requirements
Create a workflow using the official Astro GitHub Pages approach.
- Add `.github/workflows/deploy.yml` (or similar).
- Configure Pages to deploy from GitHub Actions.
- If the repo is deployed at `https://<user>.github.io/<repo>/`, set Astro’s `base` appropriately.
- If the repo is `https://<user>.github.io/` (user site repo named `<user>.github.io`), base should be `/`.

Document which base path assumption you implemented and where to change it.

## What to report at the end
When you finish, provide:
- A short summary of what you built/changed.
- The exact commands you ran and their outcomes (copy the final lines of build output).
- Clear GitHub steps to enable Pages (Settings → Pages → Source: GitHub Actions).
- Any TODOs (e.g., add headshot, replace placeholder links), but do not block on TODOs if the site builds and deploys.

## Non-goals / constraints
- Do not require API keys or secrets.
- Do not add analytics unless explicitly asked.
- Do not add a backend/contact form that requires a server; use `mailto:` or a static solution.
- Do not invent publications, awards, or credentials.

# If anything ambiguous
Please ask me questions, consult me or give me some choices and wait for my responses.

---
> Source: [rohand24/rohand24.github.io](https://github.com/rohand24/rohand24.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
