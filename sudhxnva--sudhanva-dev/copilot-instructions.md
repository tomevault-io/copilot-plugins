## sudhanva-dev

> Personal portfolio website for Sudhanva Manjunath.

# sudhanva.dev — Claude Code Guide

## Project

Personal portfolio website for Sudhanva Manjunath.
**Stack:** Next.js 15, TypeScript, Tailwind v4, motion@12 (Framer Motion)
**Theme:** Apple minimalism × Pokemon Emerald × terminal/hacker aesthetic

> **Sub-agents: read [`PLAN.md`](./PLAN.md) before starting work.** It contains the full design spec — color palette, typography, all UI patterns with exact CSS, animation strategy with timing, component descriptions, content, and a verification checklist.

## Development Workflow

### Issue-Driven Sub-Agent PRs
Each feature is tracked as a GitHub issue. Sub-agents implement features and raise PRs against `master`.

**Workflow per issue:**
1. Claim the issue (assign to yourself)
2. Create a branch: `git checkout -b feat/issue-<N>-<slug>`
3. Implement the feature described in the issue
4. Run the **Screenshot Testing Loop** (see below) — fix any visual issues found
5. Ensure `npx next build` passes with zero errors
6. Raise a PR against `master` referencing `Closes #<N>`

**Active issues:** https://github.com/sudhxnva/sudhanva-dev/issues

### Screenshot Testing Loop (Standard — 2-pass)

Every sub-agent **must** run a 2-pass visual verification before committing. Playwright + Chromium are installed as dev dependencies.

**Port assignment** (avoid collisions when agents run in parallel):
| Issue | Port |
|-------|------|
| #3 | 3003 |
| #4 | 3004 |
| #5 | 3005 |
| #6 | 3006 |
| #7 | 3007 |
| #8 | 3008 |
| #9 | 3009 |

**Steps:**
```bash
# 1. Start dev server on assigned port (background)
npm run dev -- --port <PORT> &
DEV_PID=$!
sleep 6  # wait for Next.js to be ready

# 2. Pass 1 — take screenshot
npx playwright screenshot http://localhost:<PORT> /tmp/ss-issue-<N>-pass1.png --full-page

# 3. Review the screenshot (Read the file — it's an image Claude can view)
# Compare against PLAN.md design spec:
#   - Correct colors (green-primary #00a651, bg-primary #050a05, etc.)
#   - Correct fonts (Press Start 2P for headings, JetBrains Mono for terminal/code)
#   - Pixel borders visible and clean
#   - Glass cards rendering correctly
#   - Animations wouldn't be captured but layout should be right
#   - No broken layout, overflow, or missing content

# 4. Fix any visual issues found

# 5. Pass 2 — confirm fixes
npx playwright screenshot http://localhost:<PORT> /tmp/ss-issue-<N>-pass2.png --full-page
# Read pass2 screenshot to verify issues are resolved

# 6. Kill dev server
kill $DEV_PID 2>/dev/null || true
```

**For primitive component issues (#3, etc.):** Temporarily add a demo render to `page.tsx` to visually test the components, screenshot it, then revert `page.tsx` to its original state before committing.

**What to check in screenshots:**
- Background is near-black (`#050a05`), not white
- Green accents are correct Pokémon Emerald green (`#00a651`)
- No layout breaks, horizontal overflow, or unstyled elements
- Section headings have `>_` prefix and correct font
- No console errors visible in the page render

### Branch Naming
```
feat/issue-1-core-ui-primitives
feat/issue-2-hooks-theme-toggle
feat/issue-3-interactive-primitives
feat/issue-4-loading-screen
feat/issue-5-hero-navbar
feat/issue-6-about-experience
feat/issue-7-projects-skills
feat/issue-8-education-contact
feat/issue-9-polish
```

## Critical Rules

### Imports
- **ALWAYS** import from `"motion/react"` — NEVER `"framer-motion"`
- Use `@/` alias for all internal imports

### Tailwind v4
- No `tailwind.config.ts` needed for basic usage
- Use `@theme` block in `globals.css` for custom tokens
- Custom CSS vars defined in `:root` in `globals.css`

### "use client" Directive
- Required on ANY component using React hooks or motion
- `app/layout.tsx` and `app/page.tsx` start as Server Components
- Add `"use client"` only at the component level where needed

### Glass Cards
- **NEVER** use `overflow: hidden` on glass card parents — breaks WebKit backdrop-filter
- Use `overflow: clip` if clipping is needed

### Pixel Borders
- Add `transform: translateZ(0)` to prevent hairline gaps on retina
- Use the `.pixel-border` CSS class (defined in globals.css) or the `<PixelBorder>` component

### Font Sizes
- Press Start 2P: **max 16px** — larger sizes are unreadable
- Subset to `latin` only

### Animations
- All Framer Motion variants live in `src/lib/animations.ts`
- Import and reuse — don't define inline variants
- Always check `useReducedMotion()` before animating

## Project Structure

```
src/
├── app/
│   ├── layout.tsx          # Root layout (Server Component)
│   ├── page.tsx            # Main page (Client Component)
│   └── globals.css         # Design system, CSS vars, keyframes
├── components/
│   ├── layout/
│   │   ├── NavBar.tsx
│   │   └── SectionWrapper.tsx
│   ├── sections/
│   │   ├── LoadingScreen.tsx
│   │   ├── Hero.tsx
│   │   ├── About.tsx
│   │   ├── Experience.tsx
│   │   ├── Projects.tsx
│   │   ├── Skills.tsx
│   │   ├── Education.tsx
│   │   └── Contact.tsx
│   ├── ui/
│   │   ├── PixelBorder.tsx
│   │   ├── TerminalWindow.tsx
│   │   ├── TypewriterText.tsx
│   │   ├── GlassCard.tsx
│   │   ├── PixelStatBar.tsx
│   │   ├── CRTOverlay.tsx
│   │   ├── PixelSprite.tsx
│   │   ├── SectionHeading.tsx
│   │   └── ThemeToggle.tsx
│   └── providers/
│       └── ThemeProvider.tsx
├── hooks/
│   ├── useTypewriter.ts
│   ├── useScrollSection.ts
│   └── useReducedMotion.ts
├── lib/
│   ├── animations.ts       # ALL Framer Motion variants
│   ├── cn.ts               # clsx + tailwind-merge
│   └── data.ts             # ALL portfolio content
└── types/
    └── index.ts
```

## Design Tokens (CSS Variables)

| Token | Value | Usage |
|-------|-------|-------|
| `--bg-primary` | `#050a05` | Page background |
| `--bg-secondary` | `#0d1117` | Card/terminal backgrounds |
| `--green-primary` | `#00a651` | Main accent, borders |
| `--green-bright` | `#39ff14` | Neon cursor, glows |
| `--green-muted` | `#1a4d2e` | Subtle borders |
| `--white` | `#f5f5f7` | Primary text |
| `--gray-400` | `#8e8e93` | Metadata text |
| `--amber` | `#ffb800` | Warnings, highlights |

## Commands

```bash
npm run dev     # Start dev server
npm run build   # Production build (must pass before PR)
npx tsc --noEmit  # Type check only
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sudhxnva) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
