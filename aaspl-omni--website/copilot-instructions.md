## website

> When the user says 'autopilot', 'auto-pilot', or 'yolo mode': proceed with ALL tasks without asking for confirmation. Do NOT enter plan mode — execute directly. Do NOT use Task sub-agents unless explicitly asked. Apply edits, run commands, and keep moving until the goal is complete or you hit a genuine blocker.

# Aanvikshiki Website — Claude Context

## Autopilot Mode
When the user says 'autopilot', 'auto-pilot', or 'yolo mode': proceed with ALL tasks without asking for confirmation. Do NOT enter plan mode — execute directly. Do NOT use Task sub-agents unless explicitly asked. Apply edits, run commands, and keep moving until the goal is complete or you hit a genuine blocker.

## CHANGELOG Rule
- **Single CHANGELOG**: `CHANGELOG.md` at the workspace root is the ONLY changelog file. Do NOT create or update changelogs in sub-project folders.
- **Always update before committing**: After implementing any code changes, update `CHANGELOG.md` under `## [Unreleased]` BEFORE committing. Include dated section headers, what was added/changed/fixed, key files and endpoints.
- **Never auto-push**: After committing, ask the user before running `git push`.

## Claude Operational Guidelines

1. **NEVER report incomplete tasks as complete** — Always verify task completion before marking done
2. **DO NOT worry about context window limits** — If needed, ask user to continue in a new session to complete the task
3. **ALWAYS keep responses brief but clear** — Concise communication, no unnecessary verbosity
4. **NEVER ask user to approve anything in auto-pilot mode** — Maintain and keep updating allowed command list in `.claude/settings.local.json`
5. **Maintain full context awareness** across component and page boundaries
6. **No unsolicited status documents** — Show status in conversation only
7. **Factual correctness is paramount** — Be factual with evidence, never infer without stating
8. **Requirement deviation alert** — ALERT user and update docs BEFORE implementing deviations from requirements
9. **Validate before declaring done** — Run `npm run build` (next build + next-sitemap) before marking any task complete. Do not report success without verification.
10. **Clarify ambiguity** — When requirements are ambiguous, ask clarifying questions rather than guessing intent.

## Project Overview
**Aanvikshiki Website** — A corporate/consulting website built with Next.js (App Router, static export), React, and Tailwind CSS. Features include a home page, services page, work/case studies, approach methodology, about page, contact form, and an "Ease" product page. Uses Next.js file-system routing with static generation, motion (Framer Motion) for animations, shadcn/ui + Radix UI for component primitives, and Sanity CMS for content management. Deployed to Cloudflare Pages.

## Primary Stack
- **Framework:** Next.js 15 (App Router, `output: 'export'`)
- **Language:** TypeScript 5.x (strict mode)
- **UI Library:** React 18
- **Styling:** Tailwind CSS 4 + shadcn/ui + Radix UI primitives
- **Animations:** Motion (Framer Motion)
- **Routing:** Next.js App Router (file-system based)
- **CMS:** Sanity (headless)
- **Icons:** Lucide React
- **SEO:** Next.js Metadata API + next-sitemap
- **Hosting:** Cloudflare Pages (static export)
- **Package Manager:** npm

Always assume this stack unless told otherwise.

## Tech Stack
| Component | Technology |
|-----------|------------|
| Language | TypeScript 5.x |
| Framework | Next.js 15 (App Router, static export) |
| UI Library | React 18 |
| Styling | Tailwind CSS 4, shadcn/ui, Radix UI |
| Animations | Motion (Framer Motion) |
| Routing | Next.js App Router |
| CMS | Sanity (headless) |
| Icons | Lucide React |
| Carousel | Embla Carousel |
| Hosting | Cloudflare Pages |

## Architecture

```
aanvikshikiWebsite/
├── src/
│   ├── index.css                # Global styles + Tailwind imports
│   ├── app/                     # Next.js App Router
│   │   ├── layout.tsx           # Root layout (fonts, metadata, Navbar, Footer, ThemeProvider)
│   │   ├── page.tsx             # Home page route
│   │   ├── not-found.tsx        # 404 page
│   │   ├── services/page.tsx    # Services route
│   │   ├── approach/page.tsx    # Approach route
│   │   ├── ease/page.tsx        # Ease route
│   │   ├── contact/page.tsx     # Contact route
│   │   ├── about/page.tsx       # About route
│   │   ├── work/page.tsx        # Work listing route
│   │   └── work/[slug]/page.tsx # Dynamic case study route (generateStaticParams)
│   ├── views/                   # Client-side page components
│   │   ├── Home.tsx             # Landing page
│   │   ├── Services.tsx         # Services listing
│   │   ├── Work.tsx             # Our work / portfolio
│   │   ├── CaseStudy.tsx        # Case study detail (props-based)
│   │   ├── Approach.tsx         # Methodology / approach page
│   │   ├── About.tsx            # About the company
│   │   ├── Contact.tsx          # Contact form
│   │   ├── Ease.tsx             # Ease product page
│   │   └── NotFound.tsx         # 404 page
│   ├── components/
│   │   ├── Navbar.tsx           # Site navigation with dropdown mega menu
│   │   ├── Footer.tsx           # Site footer
│   │   ├── theme-provider.tsx   # next-themes wrapper
│   │   ├── hooks/               # Custom React hooks
│   │   ├── sections/            # Reusable page sections
│   │   └── ui/                  # shadcn/ui primitives (button, card, tabs, etc.)
│   └── lib/
│       ├── caseStudies.ts       # Case study data (static fallback)
│       ├── sanity.ts            # Sanity CMS client
│       ├── sanity-queries.ts    # GROQ query functions
│       └── utils.ts             # Utility functions (cn helper)
├── sanity/schemas/              # Sanity CMS schema definitions
├── public/                      # Static assets + Cloudflare _headers
├── next.config.ts               # Next.js config (output: 'export')
├── postcss.config.mjs           # Tailwind CSS v4 PostCSS plugin
├── next-sitemap.config.js       # Sitemap generation config
├── tsconfig.json                # TypeScript configuration
├── package.json                 # Dependencies and scripts
└── CHANGELOG.md                 # Project changelog
```

### Key Patterns
- **App Router with static export**: Each route is a `page.tsx` in `src/app/` exporting metadata + importing a client component from `src/views/`
- **Shared layout**: `layout.tsx` wraps all routes with Navbar, Footer, and ThemeProvider
- **SEO via Metadata API**: Per-page `metadata` exports baked into static HTML at build time (not JS-injected)
- **Dynamic routes**: Case studies use `generateStaticParams` to pre-render all slug pages
- **Client components**: All view/section files use `'use client'` directive for motion animations
- **shadcn/ui components**: Reusable UI primitives in `src/components/ui/`
- **Section components**: Reusable page sections in `src/components/sections/`
- **Case study data**: Static fallback in `src/lib/caseStudies.ts`, CMS queries in `src/lib/sanity-queries.ts`
- **Path aliases**: `@/` maps to `src/` via Next.js and TypeScript config
- **Sanity CMS**: Schema definitions in `sanity/schemas/`, client + queries in `src/lib/`

## Important Rules

### Accuracy Rules
- Never fabricate data counts or metrics. When reporting numbers, always verify by running the actual command first.
- When modifying exports or component interfaces, verify all consuming pages/components still work.

### TypeScript/React Rules
- All components use functional component style with TypeScript
- Props interfaces should be defined explicitly, not inlined
- Use `cn()` utility from `@/lib/utils` for conditional class merging
- shadcn/ui components live in `src/components/ui/` — do not modify their core API
- Prefer composition over prop drilling for complex component trees
- Use React Router's `Link` component for internal navigation, never `<a href>`

### Styling Rules
- Tailwind CSS 4 is used — utility-first approach, no separate CSS modules
- Follow existing color scheme and design tokens from `src/lib/theme.tsx`
- Responsive design is mandatory — mobile-first approach
- Animation via `motion` library — keep animations subtle and performant

## Commands

```bash
npm run dev          # Dev server (Next.js)
npm run build        # Production build (next build + next-sitemap)
npm run start        # Serve production build locally
```

## Workflow Conventions

### Git Operations
After completing any feature or fix, automatically stage and commit with a descriptive message using conventional commit format (feat:, fix:, docs:, refactor:, test:, chore:). Do NOT auto-push — ask the user before running `git push`.

### Branch Naming
- Feature branches: `feat/<description>`, `fix/<description>`, `chore/<description>`
- Always create feature branches from `main`

### Validation
- Type check + build + sitemap: `npm run build`
- Always verify the build succeeds before committing
- Check `out/` directory for correct static HTML generation

---

## Claude Memory: Change Log, Resolution Log & TODO List

Claude maintains **three persistent documents** in its memory (`memory/`) that are updated continuously throughout the project. These are single, growing files — not per-entry scattered files.

### 1. Change Log (`memory/changelog.md`)

A single append-only log of all significant changes made to the project.

- **When to update**: After every commit-worthy change — code, config, schema, infrastructure, or convention changes
- **Claude MUST** append a new entry after completing any feature, fix, refactor, config change, or infrastructure update
- **Entry format** (append to end of file, separated by `---`):
  ```
  ### YYYY-MM-DD — <short title>
  **Scope:** <module/component affected>
  **Change:** <what was done>
  **Reason:** <why — ticket, bug, user request, requirement>
  **Supersedes:** <what previous behavior/config this replaces, or "N/A">
  **Assumptions:** <any assumptions made, or "None">
  **Files:** <key files modified>
  ```

### 2. Resolution Log (`memory/resolution_log.md`)

A single append-only log of all detected conflicts and their resolutions.

When Claude detects **conflicting information** — between memory and current code, between two memory entries, between user instructions and existing conventions, or between CLAUDE.md and actual project state — Claude MUST:
1. **STOP and report the conflict** to the user before proceeding
2. After resolution, **append** an entry to the resolution log

- **Entry format** (append to end of file, separated by `---`):
  ```
  ### YYYY-MM-DD — <short title>
  **Conflict:** <the two or more conflicting pieces of information>
  **Detected via:** <how it was found — memory vs code, memory vs memory, etc.>
  **Resolution:** <what was decided>
  **Decided by:** <user / Claude>
  **Updated:** <what was changed to eliminate the conflict — files, memory entries, CLAUDE.md>
  ```

### 3. TODO List (`memory/todo_list.md`)

A single persistent task tracker with three sections: **Active**, **Blocked**, **Completed**.

- **When to update**:
  - When the user explicitly adds, removes, or reprioritizes a task
  - When a task is completed during a session — mark it done with the completion date
  - When a new blocker or dependency is discovered
  - When the user says "add to TODO", "remind me later", or "we need to do X" — add immediately
  - At session start, if the user asks "what's pending?" or "where were we?" — read and present it
- **Entry format**:
  ```
  ## Active
  - [ ] <task description> — _added YYYY-MM-DD_ | _priority: high/medium/low_ | _scope: <module>_

  ## Blocked
  - [ ] <task description> — _added YYYY-MM-DD_ | _blocked by: <reason>_

  ## Completed
  - [x] <task description> — _added YYYY-MM-DD_ | _done YYYY-MM-DD_
  ```
- **Rules**:
  - Items stay in **Active** until done or explicitly removed by the user
  - Move to **Blocked** when a dependency/blocker is identified; note the reason
  - Move to **Completed** with the completion date when finished
  - Never silently drop a TODO — if removing, log why

### Conflict Detection Protocol

Claude MUST check for conflicts in these scenarios:
- **Before acting on a memory**: verify the remembered state still matches current code/config
- **When a user instruction contradicts CLAUDE.md**: flag it, do not silently override
- **When two memory entries disagree**: surface both, ask user to resolve
- **When a new change would invalidate an existing memory**: update or remove the stale memory
- **When git history contradicts a memory**: trust git, update the memory, log the resolution

### Housekeeping
- Change log and resolution log are append-only — do not delete entries
- Prune completed TODO items older than 30 days to keep the list focused
- Keep MEMORY.md index updated with pointers to all three documents

---

## Documentation Reference
- [Structure Overview](structure%20of%20website) — Website structure planning docs
- [Design Assets](docs/) — Design reference documents

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aaspl-omni) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
