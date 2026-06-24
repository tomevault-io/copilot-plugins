## ylh-portfolio

> Personal portfolio for a senior mobile developer (iOS + Flutter) based in Kobe, Japan. Single-page site with six sections: Header, Hero, Skills, Experience, Projects, Contact.

# Ye Lynn Htet — Portfolio

Personal portfolio for a senior mobile developer (iOS + Flutter) based in Kobe, Japan. Single-page site with six sections: Header, Hero, Skills, Experience, Projects, Contact.

## Tech Stack

| Layer | Choice | Version |
|-------|--------|---------|
| Framework | Next.js (App Router) | 16.2.9 |
| UI | React | 19.2.4 |
| Styling | Tailwind CSS | v4 (no tailwind.config) |
| Language | TypeScript | 5.x |
| Fonts | Geist Sans + Geist Mono (next/font/google) | — |
| Lint | ESLint with eslint-config-next | 9.x |

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

## Architecture

- **App Router only** — no Pages Router. All routes live under `src/app/`.
- **Single-page layout** — sections scroll on the same route (`/`). Navigation uses anchor IDs (`#skills`, `#experience`, `#projects`, `#contact`).
- **Server Components by default** — add `"use client"` only when you need interactivity (state, effects, event handlers). The site is fully static.
- **No API routes yet** — all data lives in `const` arrays and objects at the top of `page.tsx`. No database, no fetch. Future API binding is planned; see `instructions.md` for the full preparation guide.
- **Detail screens planned** — each section will have a dedicated `/skills`, `/experience`, `/projects` route. Individual projects get `/projects/[slug]` detail pages.
- **Home screen: 3-item preview** — the landing page shows 3 items per section with an "Explore More →" button linking to the full detail route. This keeps the home page scannable while offering depth.
- **Mobile-first responsive** — all Tailwind classes start at the smallest breakpoint and scale up (`sm:`, `md:`, `lg:`).

## Design System

### Colors
| Token | Class | Hex | Usage |
|-------|-------|-----|-------|
| Background | `bg-slate-950` | #020617 | Full-page canvas |
| Card surface | `bg-slate-900/60` | — | Cards, skill groups, contact footer |
| Body text | `text-slate-400` | #94a3b8 | Descriptions, secondary |
| Headings | `text-slate-50` | #f8fafc | h1, h2, h3 titles |
| Primary accent | `indigo-400/500` | #6366f1 / #818cf8 | Hero, section labels, CTA, header |
| iOS accent | `sky-400/500` | #0ea5e9 | Swift/UIKit/SwiftUI groups, SPOTV project |
| Growth accent | `emerald-400/500` | #10b981 | Tools group, Binary Lab, Rehab System |
| Warm accent | `amber-400/500` | #f59e0b | Flutter/Dart, Kakely, Light Idea |
| Streaming accent | `violet-400/500` | #8b5cf6 | SAYA project |
| POS accent | `rose-400/500` | #f43f5e | Flash Mall project |
| Borders | `border-slate-700/800` | — | Cards, tags, sections, buttons |
| Muted labels | `text-slate-500/600` | — | Section eyebrows, label text |
| Code-line decor | `h-px bg-indigo-500/40` | — | `<SectionLabel>` left decoration |

### Typography
- **Display/headings**: Geist Sans (`--font-geist-sans`), bold, tight tracking
- **Body**: Geist Sans, regular weight, relaxed leading
- **Utility/mono**: Geist Mono (`--font-geist-mono`), uppercase tracking-widest, used for section eyebrow labels and logo label text

### Accent token system
All accents are defined in the `accentTokens` record at the top of `page.tsx`. Each accent has three slots:

```ts
type Accent = "indigo" | "sky" | "emerald" | "amber" | "violet" | "rose";

const accentTokens: Record<Accent, { bar: string; tag: string; glow: string }> = {
  indigo:  { bar: "bg-indigo-500",  tag: "text-indigo-400  border-indigo-500/20",  glow: "bg-indigo-500/10" },
  // ...etc
};
```

To add a new accent: add an entry to `Accent` union + `accentTokens`. Tag classes use `border-<color>-500/20` for a muted border that matches the accent.

### Reusable micro-components (all in `page.tsx`)
- `<SectionLabel>` — mono eyebrow label with indigo code-line decoration (`h-px w-8`)
- `<AccentBar accent>` — small colored bar (`h-1 w-8 rounded-full`)
- `<Tag accent>` — rounded pill badge with accent border + text + `bg-slate-900/80`
- `<HeroIllustration />` — orbiting logo ring (8 tech logos on circular track with slow rotation, speeds up on hover)

### Hero illustration: logo orbit
8 technology logos arranged on a circular orbit track. The outer ring rotates clockwise via `animate-orbit` (24s), each logo icon counter-rotates via `animate-unorbit` to stay upright. On `group-hover` both switch to `-fast` variants (8s). Logos use semi-transparent fills to blend with the background. Three concentric ghost rings provide depth. Logos positioned using pre-computed `translate(cos(angle)*R, sin(angle)*R)` offsets on a 112px radius.

### Section patterns
| Section | Pattern | Key classes |
|---------|---------|-------------|
| Header | Sticky nav with backdrop-blur | `sticky top-0 z-50 border-b border-slate-800/50 bg-slate-950/80 backdrop-blur-md` |
| Hero | Two-column (text left, orbit right) | `flex flex-col sm:flex-row gap-10 sm:gap-16` |
| Skills | 4 skill-group cards in 2-col grid | Grid + `<AccentBar>` + `<Tag>` list |
| Experience | Vertical timeline with dots | `<ol>` + absolute line + timeline dots + card content |
| Projects | 5 cards in responsive grid | `grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3` + hover lift + glow + real App Store/GitHub links |
| Contact | Footer card with contact grid | `<dl>` grid + `<SectionLabel>` + copyright — Location: Kobe, GitHub: ye-lynn-htet |

### Card base style
`rounded-xl border border-slate-800 bg-slate-900/60 p-5 sm:p-6 transition-all hover:border-slate-700`

### Animation system (all in `globals.css`)
| Animation | Duration | Behavior |
|-----------|----------|----------|
| `animate-orbit` | 24s linear infinite | Full clockwise rotation |
| `animate-orbit-fast` | 8s linear infinite | Speed-up on group hover |
| `animate-unorbit` | 24s linear infinite | Counter-rotation (keeps icons upright) |
| `animate-unorbit-fast` | 8s linear infinite | Counter-rotation speed-up |

### Responsive breakpoints
| Breakpoint | Width | What changes |
|-----------|-------|-------------|
| Base (mobile) | <640px | 1-col grid, `text-4xl` h1, compact nav labels (first 3 chars), reduced padding/gap, orbit 72 → 80% |
| `sm:` | ≥640px | 2-col grid, `text-5xl` h1, row layout for hero, full nav labels, normal padding/gap, orbit 80% |
| `md:` | ≥768px | `text-6xl` h1 |
| `lg:` | ≥1024px | 3-col project grid, `text-6xl` h1 |

## Conventions

- **Mobile-first**: every Tailwind utility scales from base → `sm:` → `md:` → `lg:`
- **Static data as `const`**: all data arrays (`navLinks`, `skillGroups`, `experiences`, `projects`, `contact`, `orbitLogos`) live at module scope, typed with `as const`
- **No import of Link unless navigated** — anchor links use `<a href="#id">`
- **Sections** use `<section>` with `id` for scroll targets and `scroll-mt-20` so sticky header doesn't cover them
- **`text-balance`** on hero paragraph for clean line breaks on narrow screens
- **`scroll-smooth`** set once on `<html>` in `layout.tsx`
- **`transition-all`** on interactive elements — consistent feel
- **Don't import Tailwind** — v4 uses `@import "tailwindcss"` in `globals.css`, no config file
- **Hero illustration is the signature element** — keep it distinct, don't clutter other sections with animation
- **Accessibility**: `aria-hidden="true"` on decorative elements, `role="img"` with `aria-label` on illustrations, `role="list"` on semantic lists, `aria-label` on landmark sections

## Commands

```bash
npm run dev        # Start dev server on port 3000
npm run build      # Production build
npm run start      # Start production server
npm run lint       # Run ESLint
```

## Constraints — DO NOT

- Add npm dependencies without asking first
- Use Pages Router patterns or `pages/` directory
- Convert to `"use client"` unless the component genuinely needs interactivity
- Break the dark theme — the project is slate-950 based, no light mode
- Remove or restructure `accentTokens` — it's the design token pattern
- Use `tailwind.config.ts` — Tailwind v4 config lives in CSS
- Remove the logo orbit or replace it lightly — it's the page's signature visual element
- Change the slate-950 base color to zinc or neutral — slate was a deliberate choice to avoid AI-default palettes
- Create separate component files without confirming — the site is co-located in `page.tsx`

## Git Workflow

After completing a code change, follow this sequence — do NOT skip any step:

1. **Verify the change** — run `npm run build` to confirm the project compiles without errors.
2. **Show the output** — present the build result to the user. If it fails, fix and repeat step 1.
3. **Wait for confirmation** — ask the user to approve before staging, committing, or pushing.
4. **Commit and push** — only after the user confirms:
   ```bash
   git add . && git commit -m "<descriptive message>" && git push
   ```

Never commit or push without the user explicitly approving it first.

## Installed Agents

| Agent | File | Use when |
|-------|------|----------|
| `frontend-ui-engineering` | `.claude/skills/frontend-ui-engineering/SKILL.md` | Building/modifying UI components, implementing layouts, managing state, production-quality output |
| `frontend-design` | `.claude/skills/frontend-design/SKILL.md` | Designing new sections, choosing visuals, typography, palette decisions |
| `vercel-react-best-practices` | `.claude/skills/vercel-react-best-practices/SKILL.md` | Writing/refactoring React or Next.js code, data fetching, bundle optimization |

All three are registered as project skills in `.claude/skills/` with corresponding agent definitions in `.claude/agents/`. Invoke them with `/frontend-ui-engineering`, `/frontend-design`, or `/vercel-react-best-practices` when their domain applies.

---
> Source: [ye-lynn-htet/ylh-portfolio](https://github.com/ye-lynn-htet/ylh-portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
