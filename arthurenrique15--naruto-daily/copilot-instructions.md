## naruto-daily

> Naruto Daily is a daily character-guessing game inspired by Narutodle. Players guess a Naruto character

# Naruto Daily — Claude Code Context

## Project Overview

Naruto Daily is a daily character-guessing game inspired by Narutodle. Players guess a Naruto character
based on attribute feedback (classic Wordle-style). One puzzle per day, determined by a deterministic
date-based seed.

## Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Next.js (TypeScript, App Router) |
| Styling | Tailwind CSS + shadcn/ui |
| Deploy | Vercel |
| Data | `data/characters.json` — static, versioned in repo |
| Scraper | Python (BeautifulSoup + requests / MediaWiki API) |
| Language | English (i18n architecture mapped for future use) |

## Project Structure

```
naruto-daily/
├── CLAUDE.md
├── README.md
├── docs/
│   ├── ARCHITECTURE.md
│   ├── DATA-SCHEMA.md
│   ├── SCRAPER.md
│   ├── GAME-MECHANICS.md
│   └── ROADMAP.md
├── data/
│   ├── characters.json          # game-ready, from filter script — DO NOT edit manually
│   ├── characters-raw.json      # scraper output (gitignored)
│   └── canon-arcs.json          # manual whitelist of canonical arcs
├── scripts/
│   └── filter-characters.ts     # TypeScript filter: raw → game-ready
├── scraper/
│   ├── main.py
│   ├── requirements.txt
│   └── README.md
├── src/
│   ├── app/                     # Next.js App Router pages
│   ├── components/              # React components
│   ├── lib/
│   │   ├── daily.ts             # daily seed logic
│   │   ├── game.ts              # attribute comparison logic
│   │   └── storage.ts           # localStorage helpers
│   └── types/
│       └── character.ts         # TypeScript types
├── public/
├── package.json
├── tsconfig.json
└── next.config.ts
```

## Essential Commands

```bash
# Development
npm run dev          # Start Next.js dev server (http://localhost:3000)
npm run build        # Production build
npm run lint         # ESLint
npm run type-check   # TypeScript strict check (tsc --noEmit)

# Scraper — IMPORTANT restrictions for agents
# NEVER run the full scraper to validate implementations
# Only run for specific characters or with a limited character count:
npm run scrape           # Full scrape → data/characters-raw.json (slow, network)
npm run filter-data      # Filter raw → data/characters.json (fast, local)
npm run build-data       # Full pipeline: scrape + filter
npm run validate-data    # Validate characters.json schema

# Agent-safe scraping (NEVER run full scrape to validate implementations):
npm run scrape-test                                        # limit 5 characters
cd scraper && python main.py --limit 5                    # equivalent
cd scraper && python main.py --characters "Naruto Uzumaki,Sasuke Uchiha"
```

## UI & React Best Practices

When writing or reviewing React/Next.js UI code, invoke the `vercel-react-best-practices` skill before implementing. Key rules to follow:
- Prefer Server Components; use `'use client'` only when necessary
- Avoid hydration mismatches — never read `localStorage`/`window` in `useState` initializers; use `useEffect`
- Load client-only state after hydration to prevent SSR/client HTML mismatch
- Use `whitespace-normal` in nested components to override ancestor `whitespace-nowrap`

## Plans

Whenever the user asks to create a plan, create a `.md` file for it in `docs/plans/` following the naming convention `YYYY-MM-DD-<feature-name>.md`. This is the default path for all implementation plans.

When executing a plan, follow every step exactly as written — including the git workflow (worktree creation, branch naming, commit order, PR creation). Do not skip or reorder steps. If a step cannot be completed, stop and explain why rather than silently skipping it.

**IMPORTANT for plan agents**: At the top of every plan file, include this instruction for the executing agent:
> Before touching any files, read `CLAUDE.md` in the project root to understand conventions, constraints, and workflow rules.

## Conventions

### TypeScript
- Strict mode enabled (`tsconfig.json`: `"strict": true`)
- All types live in `src/types/`
- No `any` — use proper types or `unknown`
- Use `interface` for data shapes, `type` for unions/aliases

### Components
- All React components in `src/components/`
- Use shadcn/ui base components (Button, Input, Badge, Dialog, etc.)
- Prefer Server Components; use `'use client'` only when necessary
- Component files: PascalCase (e.g., `GuessRow.tsx`)
- Avoid nested conditionals in render logic — extract each case into a named sub-component or helper function with early returns

### Data
- **NEVER modify `data/characters.json` manually** — regenerate via scraper
- **NEVER add filler-only characters** — only characters from canonical manga arcs
- Canonical arc whitelist: `data/canon-arcs.json`
- **Franchise scope**: only Naruto Part I and Naruto Shippuden (manga ch 1-700). Boruto, movie-only, and filler-only characters are excluded. The `series` field in `canon-arcs.json` enforces this at scraper load time.
- All string values in English, lowercase-normalized where applicable
- Array fields (e.g., `naturTypes`, `jutsuTypes`) must be sorted alphabetically

### Styling
- Tailwind utility classes only — no custom CSS files unless unavoidable
- Dark mode supported via Tailwind `dark:` prefix
- Feedback colors: green = correct, yellow = partial, gray = wrong

### Git
- Branch naming: `type/description-in-kebab-case` (e.g. `feat/web-scraper`, `fix/seed-calculation`, `chore/update-scraper-docs`)
- Commits: conventional commits format (`feat: add classic mode`, `fix: seed calculation`)

## Branch & PR Workflow

Use the **git-workflow** skill for all feature/fix work. It covers worktree creation,
branch naming, commit conventions, push, and PR creation.

Key rules:
- Worktrees go in `worktrees/{branch-name}` inside the repo
- Branch naming: `type/description-in-kebab-case`
- Commits: conventional format, one per logical change
- Always follow every step in order — do not skip

**Mandatory safety rules:**
- **NEVER touch files in the main repository root** unless explicitly instructed to work directly in the main repo
- **NEVER commit anything on the `main` branch** — all work happens on feature branches in worktrees
- **ALWAYS use git worktrees and the git-workflow skill** for every implementation task

## Review Agent

When reviewing a pull request, use the Superpowers **requesting-code-review** skill:

### 1. Load the skill
```
mcp_superpowers_read_skill(skill_name: "requesting-code-review")
```

### 2. Get PR context
```bash
# When on the PR branch:
BASE_SHA=$(git rev-parse origin/main)   # or the PR's base branch
HEAD_SHA=$(git rev-parse HEAD)         # PR branch tip
```

### 3. Run code review
Follow the skill workflow. Provide:
- **WHAT_WAS_IMPLEMENTED**: Summary of PR changes
- **PLAN_OR_REQUIREMENTS**: What the feature/PR should do (from PR description or linked issue)
- **BASE_SHA** / **HEAD_SHA**: Commit range to review
- **DESCRIPTION**: Brief summary

### 4. Add comments to the PR — MANDATORY, never skip this step

**After the code review subagent returns its findings, you MUST immediately post the review to the PR via `gh api`. Do not just summarize the results in the chat — post them to GitHub.**
Use `gh api` to post a formal PR review with inline comments (not just a plain comment):

```bash
gh api repos/{owner}/{repo}/pulls/{PR_NUMBER}/reviews \
  --method POST \
  --field commit_id="<HEAD_SHA>" \
  --field event="COMMENT" \
  --field body="## Code Review — by Claude
  ...overall summary...
  🤖 *Made by Claude (via Claude Code)*" \
  --field "comments[][path]=path/to/file.py" \
  --field "comments[][line]=<line_number>" \
  --field "comments[][body]=**Severity: issue description**
  ...details...
  🤖 *Made by Claude (via Claude Code)*"
```

**Rules:**
- Always use `event="COMMENT"` — GitHub does not allow `REQUEST_CHANGES` on your own PRs
- Every comment (overall body and each inline comment) must end with `🤖 *Made by Claude (via Claude Code)*` to identify the author
- Prefer inline comments on specific lines over generic PR-level comments
- Post inline comments alongside the overall review in a single API call (use multiple `--field "comments[]..."` entries)

### 5. Severity handling
- **Critical**: Must fix before merge
- **Important**: Fix before proceeding
- **Minor**: Note for later; optional to fix

### 6. Resolving PR comments
When addressing review feedback:
- **Commit each change separately** — one logical fix per commit (e.g. `fix: add rate limiting to resolve_image_url`)
- **Push after all changes** — push the branch when done
- **Resolve all comments** — mark each PR comment as resolved on GitHub
- **Reply when needed** — if a resolution is not valid or needs more explanation, reply to the comment with context
- **Reply identifier** — always end replies with `🤖 *Made by Claude (via Claude Code)*` to identify the author

## Key Design Decisions

- **Daily puzzle**: `daysSinceEpoch % totalCharacters` — deterministic, no backend needed
- **Data at build time**: characters loaded via RSC / `getStaticProps` — no client-side data fetching
- **Filler filter**: characters appearing only in non-canonical arcs are excluded at scraper level
- **i18n**: architecture ready (next-intl), but English-only in MVP

## Documentation

- `docs/ARCHITECTURE.md` — system architecture and data flow
- `docs/DATA-SCHEMA.md` — `characters.json` schema and field conventions
- `docs/SCRAPER.md` — scraper design, data sources, how to run
- `docs/GAME-MECHANICS.md` — game rules, feedback system, state management
- `docs/ROADMAP.md` — phased implementation plan and future modes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ArthurEnrique15) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
