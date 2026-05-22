## my-portfolio

> > This file is the single source of truth for Claude Code working on this project.

# CLAUDE.md — Portfolio Project

> This file is the single source of truth for Claude Code working on this project.
> Read this before touching any file. Follow every rule here strictly.

---

## Project Overview

Personal portfolio website for **Muhammad Jamil Raza Attari**, an AI/ML Engineer,
Data Scientist, and Educator based in Karachi, Pakistan.

**Primary goals:**
- Attract international remote AI/ML job opportunities
- Convert freelance clients (Upwork / Fiverr)
- Build personal brand as an AI practitioner and educator

**Core identity tagline:**
> "I Build Intelligent AI Systems — and Teach Others to Do the Same"

**Audiences (in priority order):**
1. International remote recruiters (AI/ML roles)
2. Local Pakistani companies (job search)
3. Freelance clients (Upwork / Fiverr)

---

## Tech Stack & Tools

| Layer        | Technology                          | Version     |
|--------------|-------------------------------------|-------------|
| Framework    | Next.js (App Router)                | 14.x latest |
| Language     | TypeScript                          | 5.x         |
| Styling      | Tailwind CSS                        | 3.x         |
| Animations   | Framer Motion                       | 11.x        |
| Icons        | Lucide React                        | latest      |
| Fonts        | next/font/google (self-hosted)      | built-in    |
| Contact Form | Formspree                           | free tier   |
| Deployment   | Vercel                              | free tier   |
| Package Mgr  | npm                                 | —           |

**Font pairs (dual-mode):**
- Light mode: `Lora` (heading) + `Outfit` (body)
- Dark mode:  `JetBrains Mono` (heading) + `DM Sans` (body)

---

## Project Structure

```
jamil-portfolio/
├── app/
│   ├── layout.tsx          ← Root layout: font variables, metadata, ThemeProvider
│   ├── page.tsx            ← Single page: all sections composed here
│   ├── globals.css         ← CSS variables, Tailwind base, dark mode overrides
│   └── fonts.ts            ← next/font declarations for all 4 fonts
│
├── components/
│   ├── layout/
│   │   ├── Navbar.tsx      ← Logo, nav links, ThemeToggle
│   │   └── Footer.tsx      ← Social links, copyright
│   ├── sections/
│   │   ├── Hero.tsx        ← Name, tagline, CTAs, profile photo
│   │   ├── About.tsx       ← Value proposition, remote-ready signal
│   │   ├── Skills.tsx      ← 4-category skill grid
│   │   ├── Projects.tsx    ← Featured 4 projects (card grid)
│   │   ├── Experience.tsx  ← Timeline: SMIT, Saylani, Deloitte, HBL, ibex
│   │   ├── Certifications.tsx ← Badge layout, top certs only
│   │   └── Contact.tsx     ← Formspree form, social icons
│   └── ui/
│       ├── ThemeToggle.tsx ← Sun/Moon toggle, localStorage persistence
│       ├── ProjectCard.tsx ← Reusable card: title, tech, links
│       ├── SkillBadge.tsx  ← Pill badge for skill tags
│       ├── TimelineItem.tsx← Single experience row
│       └── SectionLabel.tsx← Eyebrow label (FEATURED PROJECTS, etc.)
│
├── data/
│   ├── projects.ts         ← All project data (title, desc, tech, links)
│   ├── skills.ts           ← Skills grouped by category
│   ├── experience.ts       ← Work/education timeline entries
│   └── certifications.ts  ← Cert name, issuer, date, badge URL
│
├── public/
│   ├── profile.jpg         ← Professional headshot (optimized, WebP preferred)
│   ├── resume.pdf          ← Downloadable CV (keep updated)
│   └── og-image.png        ← Open Graph image for social sharing (1200×630)
│
├── lib/
│   └── utils.ts            ← cn() helper (clsx + tailwind-merge)
│
├── tailwind.config.ts
├── tsconfig.json
├── next.config.ts
├── .env.local              ← FORMSPREE_ID (never commit this)
└── CLAUDE.md               ← This file
```

---

## Design System

### Color Tokens

All colors are defined as CSS variables in `globals.css`.
**Never hardcode hex values in components — always use token names.**

```css
/* Light mode — :root */
--color-bg:         #FAFAFA;    /* Page background           */
--color-surface:    #F3F4F8;    /* Cards, skill row bg        */
--color-accent:     #1740D4;    /* CTAs, links, badges        */
--color-accent-sub: #EEF0FF;    /* Accent background tint     */
--color-ink:        #080918;    /* Primary text               */
--color-muted:      #44445A;    /* Secondary text, subtitles  */
--color-border:     #E8E8EF;    /* Dividers, card borders     */
--color-card:       #FFFFFF;    /* Project card background    */

/* Dark mode — .dark */
--color-bg:         #0C0C16;
--color-surface:    #101018;
--color-accent:     #05D4B4;
--color-accent-sub: #0D2420;
--color-ink:        #F0F0FA;
--color-muted:      #8888AA;
--color-border:     #1E1E2E;
--color-card:       #101018;
```

### Tailwind Mapping

```ts
// tailwind.config.ts — extend.colors
colors: {
  bg:      'var(--color-bg)',
  surface: 'var(--color-surface)',
  accent:  'var(--color-accent)',
  'accent-sub': 'var(--color-accent-sub)',
  ink:     'var(--color-ink)',
  muted:   'var(--color-muted)',
  border:  'var(--color-border)',
  card:    'var(--color-card)',
}
```

### Typography

```ts
// tailwind.config.ts — extend.fontFamily
fontFamily: {
  heading: ['var(--font-heading)'],  // Lora (light) / JetBrains Mono (dark)
  body:    ['var(--font-body)'],     // Outfit (light) / DM Sans (dark)
}
```

**Type scale:**

| Role              | Class                                   |
|-------------------|-----------------------------------------|
| Hero name         | `font-heading text-5xl md:text-7xl font-medium tracking-tight` |
| Section heading   | `font-heading text-3xl font-medium`     |
| Card title        | `font-heading text-lg font-medium`      |
| Body text         | `font-body text-base leading-relaxed`   |
| Eyebrow label     | `font-body text-[9px] tracking-[2.5px] uppercase` |
| Skill badge       | `font-body text-[10px] tracking-[0.3px]` |

### Spacing Scale

Base unit: `4px` (Tailwind default).
Use only multiples: `4, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80, 96`.

**Section padding:** `py-24 md:py-32`
**Container:** `max-w-5xl mx-auto px-6`
**Card padding:** `p-5 md:p-6`
**Gap between cards:** `gap-4 md:gap-6`

### Border Radius

| Element       | Class          |
|---------------|----------------|
| Buttons       | `rounded`      |
| Cards         | `rounded-xl`   |
| Badges/pills  | `rounded-sm`   |
| Avatar/photo  | `rounded-2xl`  |
| Toggle pill   | `rounded-full` |

---

## Coding Conventions

### File & Component Naming

```
Components   → PascalCase         e.g. ProjectCard.tsx
Hooks        → camelCase, use*    e.g. useTheme.ts
Utilities    → camelCase          e.g. formatDate.ts
Data files   → camelCase          e.g. projects.ts
CSS classes  → kebab-case         (Tailwind utility only — no custom classes)
```

### Component Pattern

```tsx
// ✅ Correct — named export, typed props, arrow function
interface ProjectCardProps {
  title: string;
  description: string;
  techStack: string[];
  githubUrl: string;
  liveUrl?: string;           // optional fields use ?
}

export const ProjectCard = ({ title, description, techStack, githubUrl, liveUrl }: ProjectCardProps) => {
  return (
    // ...
  );
};
```

### Import Order (enforced by ESLint)

```ts
// 1. React / Next.js core
import { useState, useEffect } from 'react';
import Image from 'next/image';
import Link from 'next/link';

// 2. Third-party libraries
import { motion } from 'framer-motion';
import { Github, ExternalLink } from 'lucide-react';

// 3. Internal components (absolute path with @/ alias)
import { ProjectCard } from '@/components/ui/ProjectCard';
import { SectionLabel } from '@/components/ui/SectionLabel';

// 4. Data imports
import { projects } from '@/data/projects';

// 5. Types
import type { Project } from '@/types';
```

### TypeScript Rules

- Use `interface` for props and data shapes, not `type`
- No `any` — use `unknown` and narrow, or define a proper interface
- All component props must be explicitly typed
- Data arrays in `data/*.ts` must have explicit return types

### Styling Rules

- **Tailwind only** — no inline `style={{}}`, no CSS modules, no styled-components
- Use `cn()` utility (clsx + tailwind-merge) for conditional classes
- Dark mode via Tailwind `dark:` prefix where CSS variables aren't sufficient
- Mobile-first always: base styles = mobile, `md:` prefix = desktop
- No `!important` overrides

```tsx
// ✅ Correct
import { cn } from '@/lib/utils';
<div className={cn('text-ink font-body', isActive && 'text-accent')} />

// ❌ Wrong
<div style={{ color: '#080918' }} />
<div className="text-[#080918]" />   // never hardcode hex
```

### Animation Rules

- Use Framer Motion for scroll-triggered reveals (section entrances)
- Keep `duration` between `0.4s` and `0.6s` — no flashy long animations
- Stagger children max `0.1s` delay between items
- `initial={{ opacity: 0, y: 20 }}` → `animate={{ opacity: 1, y: 0 }}` is the standard reveal
- No animations on mobile unless performance tested — use `useReducedMotion()`

```tsx
// Standard section reveal pattern
const containerVariants = {
  hidden: {},
  visible: { transition: { staggerChildren: 0.1 } }
};
const itemVariants = {
  hidden:  { opacity: 0, y: 20 },
  visible: { opacity: 1, y: 0, transition: { duration: 0.5, ease: 'easeOut' } }
};
```

### Image Rules

- Always use `next/image` — never `<img>` tag
- Profile photo: `priority` prop (above the fold)
- All images must have descriptive `alt` text
- Use WebP format where possible

---

## Theme System

Dark/light mode is controlled by a `.dark` class on `<html>`.

**Rule:** Never use `dark:text-white` style Tailwind dark prefixes for color tokens.
The CSS variables in `:root` vs `.dark` handle all color switching automatically.
Only use `dark:` prefix for structural differences (e.g., `dark:border-opacity-50`).

**Font switching is automatic** via CSS variables:
```css
:root  { --font-heading: var(--font-heading-light); }  /* Lora          */
.dark  { --font-heading: var(--font-heading-dark);  }  /* JetBrains Mono */
```

**ThemeToggle persistence:** `localStorage.getItem('theme')` on mount.
Fallback: `prefers-color-scheme` media query.
Flash prevention: Add inline script in `<head>` before hydration.

---

## Data Structure

### projects.ts

```ts
export interface Project {
  id:          string;
  title:       string;
  category:    string;       // e.g. 'RAG · CHATBOT'
  description: string;       // one sentence — problem + impact
  techStack:   string[];
  githubUrl:   string;
  liveUrl?:    string;       // optional — only if deployed
  featured:    boolean;      // true = shown on homepage (max 4)
}
```

### Featured projects (in display order):

1. Broadway Pizza Chatbot — `featured: true` — has liveUrl
2. Pothole Detection YOLOv8 — `featured: true` — githubUrl only
3. SpaceX Launch Prediction — `featured: true` — has liveUrl
4. Heart Disease Dashboard — `featured: true` — githubUrl only

Real Estate and Secure Encryption — `featured: false` (GitHub only, not on homepage).

---

## Git Commit Style

Follow **Conventional Commits** strictly.

```
<type>(<scope>): <short description>

Types:
  feat      ← new section, new component, new feature
  fix       ← bug fix, layout issue, broken link
  style     ← CSS/Tailwind changes, spacing, colors
  refactor  ← code restructure, no behavior change
  content   ← copy edits, project data updates, resume update
  perf      ← image optimization, lazy loading, bundle size
  a11y      ← accessibility improvements
  docs      ← README, CLAUDE.md updates
  chore     ← config changes, dependency updates
```

**Examples:**

```
feat(hero): add animated tagline with framer motion
feat(projects): add broadway pizza chatbot card
fix(navbar): mobile menu z-index overlap on scroll
style(hero): adjust heading size for sm breakpoint
content(projects): update spacex project description and links
perf(images): convert profile photo to webp format
a11y(contact): add aria-label to social icon buttons
chore(deps): update framer-motion to v11
```

**Branch naming:**
```
feature/hero-section
feature/projects-grid
fix/mobile-nav
content/update-experience
```

---

## Commands

```bash
# Development
npm run dev          # Start dev server → localhost:3000
npm run build        # Production build
npm run start        # Run production build locally
npm run lint         # ESLint check
npm run type-check   # TypeScript check (tsc --noEmit)

# Deployment
# Push to main branch → Vercel auto-deploys
# Preview: every PR gets a unique preview URL from Vercel

# Utilities
npm run lint -- --fix        # Auto-fix lint errors
```

### Environment Variables

```bash
# .env.local (never commit — add to .gitignore)
NEXT_PUBLIC_FORMSPREE_ID=your_form_id_here
```

---

## SEO & Metadata

```tsx
// app/layout.tsx — fill these in
export const metadata: Metadata = {
  title: 'Muhammad Jamil Raza Attari — AI/ML Engineer',
  description: 'AI/ML Engineer specialized in Agentic AI, RAG systems, and Computer Vision. Based in Karachi, open to remote roles and freelance.',
  keywords: ['AI Engineer', 'ML Engineer', 'RAG', 'LangChain', 'Agentic AI', 'Computer Vision', 'Python', 'Karachi'],
  openGraph: {
    title: 'Muhammad Jamil Raza Attari — AI/ML Engineer',
    description: 'Building intelligent AI systems and teaching others to do the same.',
    url: 'https://your-domain.vercel.app',
    siteName: 'Jamil Raza Portfolio',
    images: [{ url: '/og-image.png', width: 1200, height: 630 }],
    locale: 'en_US',
    type: 'website',
  },
  twitter: {
    card: 'summary_large_image',
    title: 'Muhammad Jamil Raza Attari — AI/ML Engineer',
    images: ['/og-image.png'],
  },
  robots: { index: true, follow: true },
};
```

---

## Performance Checklist

- [ ] All images use `next/image` with explicit `width` and `height`
- [ ] Profile photo uses `priority` prop
- [ ] Fonts loaded via `next/font` (self-hosted, no external requests)
- [ ] Framer Motion: only animate above-the-fold section eagerly; rest use `whileInView`
- [ ] `useReducedMotion()` check before running animations
- [ ] No unused dependencies in `package.json`
- [ ] Lighthouse score target: Performance ≥ 90, Accessibility ≥ 95

---

## Accessibility Rules

- All `<img>` and `<Image>` must have descriptive `alt`
- Icon-only buttons must have `aria-label`
- Color contrast: text on bg must pass WCAG AA (4.5:1 minimum)
- Focus states must be visible — never `outline: none` without a replacement
- Semantic HTML: use `<section>`, `<nav>`, `<main>`, `<footer>` correctly
- Each section must have a heading (`h2`) for screen readers

---

## Do's and Don'ts

### ✅ Do

- Mobile-first CSS — base = mobile, `md:` = desktop
- Use `cn()` for all conditional classNames
- Keep all content/copy in `data/*.ts` — never hardcode in JSX
- Use `next/image` for every image
- Use `next/link` for every internal link
- Test toggle: both dark and light mode after every UI change
- Run `npm run type-check` before every commit
- Verify on Chrome + Firefox + Safari + mobile viewport (375px)
- Keep `CLAUDE.md` updated when stack or conventions change

### ❌ Don't

- Never hardcode hex colors in components — use token variables
- Never use inline `style={{}}` for colors or fonts
- Never use `<img>` — always `next/image`
- Never use `<a href>` for internal routes — always `next/link`
- Never commit `.env.local`
- Never add a project to the homepage without `featured: true` and a real description
- Never skip TypeScript types — no `any`
- Never use `// @ts-ignore` — fix the actual type issue
- Never add animations without testing on a low-end device simulation
- Never leave placeholder text (Lorem ipsum, TODO, [YOUR NAME]) in production
- Never use deprecated Next.js patterns (`pages/` router, `getServerSideProps`, `_app.tsx`)

---

## Contact Form Setup (Formspree)

1. Go to formspree.io → create free account
2. Create new form → copy the form ID
3. Add to `.env.local`: `NEXT_PUBLIC_FORMSPREE_ID=xyzabc12`
4. Use `@formspree/react` package or plain fetch POST

```tsx
// Simple fetch approach (no extra package needed)
const response = await fetch(`https://formspree.io/f/${process.env.NEXT_PUBLIC_FORMSPREE_ID}`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ name, email, message }),
});
```

---

## Sections Checklist

| Section        | Component          | Status |
|----------------|--------------------|--------|
| Navbar         | Navbar.tsx         | [x]    |
| Hero           | Hero.tsx           | [ ]    |
| About          | About.tsx          | [ ]    |
| Skills         | Skills.tsx         | [ ]    |
| Projects       | Projects.tsx       | [ ]    |
| Experience     | Experience.tsx     | [ ]    |
| Certifications | Certifications.tsx | [ ]    |
| Contact        | Contact.tsx        | [ ]    |
| Footer         | Footer.tsx         | [x]    |
| Theme Toggle   | ThemeToggle.tsx    | [x]    |

---

## Known Constraints (discovered during build)

### lucide-react — No Brand Icons
The installed version of `lucide-react` does **not** export `Github` or `Linkedin`.
Use inline SVG for these two icons wherever needed (Footer, Contact section).
All other icons (`Mail`, `MapPin`, `Sun`, `Moon`, `Menu`, `X`, `ChevronDown`, etc.) work normally.

```tsx
// ✅ Use this pattern for GitHub/LinkedIn icons
const IconGithub = () => (
  <svg width="18" height="18" viewBox="0 0 24 24" fill="currentColor" aria-hidden="true">
    <path d="M12 0C5.37 0 0 5.37 0 12c0 5.31 3.435 9.795 8.205 11.385..." />
  </svg>
);
// Copy exact SVG from components/layout/Footer.tsx
```

### CSS Color Variables — RGB Channel Format
Color variables in `globals.css` use space-separated RGB channels (not hex):
```css
--color-bg: 250 250 250;   /* NOT #FAFAFA */
```
Tailwind config uses `rgb(var(--color-bg) / <alpha-value>)` format.
This enables opacity modifiers: `bg-bg/80`, `border-accent/40`, `text-muted/60`.
**Do not revert to hex format** — opacity modifiers will break.

### LinkedIn Profile URL
Correct URL: `https://www.linkedin.com/in/jamilrazaa`

---

*Last updated: Phase 2 complete — May 2026*
*Maintainer: Muhammad Jamil Raza Attari*

---
> Source: [JamilRaza001/my-portfolio](https://github.com/JamilRaza001/my-portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
