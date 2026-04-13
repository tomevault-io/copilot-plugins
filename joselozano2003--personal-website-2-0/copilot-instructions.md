## personal-website-2-0

> When asked to **sync**, **update**, or **refresh** the portfolio content, do the following:

# Portfolio — Claude Instructions

## Syncing with Obsidian Profile

When asked to **sync**, **update**, or **refresh** the portfolio content, do the following:

1. Run `obsidian read file="Jose Lozano - Full Profile"` to get the latest version of the profile.
2. Compare the content against what is currently rendered in the components.
3. Update only the components that have changed — do not touch layout, styling, or design.

### What to sync and where

| Profile section | Component file |
|---|---|
| Personal info (location, email, phone) | `components/contact.tsx` and `components/hero.tsx` |
| Education | `components/about.tsx` (details row) |
| Professional Experience | `components/experience.tsx` |
| Projects | `components/projects.tsx` |
| Technical Skills | `components/skills.tsx` |
| Leadership & Extracurriculars | `components/leadership.tsx` |
| Motivations / tagline copy | `components/hero.tsx` and `components/about.tsx` |

### Rules

- **Only update content** (text, bullets, tech stacks, dates, numbers). Never change colors, fonts, spacing, or component structure unless explicitly asked.
- If a new experience or project is added in the Obsidian file, add a new card following the exact same pattern as existing ones.
- If an experience or project is removed from the Obsidian file, remove the corresponding card.
- If the tech stack for a skill group changes, update the array in `components/skills.tsx`.
- Preserve all existing copy tone — professional, direct, no em dashes.
- After syncing, run `npm run build` to confirm no errors before finishing.

## Project location

`/Users/joselozano2003/Desktop/Projects/portfolio`

Dev server: `npm run dev` — runs on port 3000.

## Stack

- Next.js 16 (App Router), TypeScript
- Tailwind v4, Shadcn UI
- Fonts: Syne (headings), JetBrains Mono (labels), Instrument Serif (italic accents)
- Color accent: `#C8956C` (terracotta-sand), background: `#F8F5F0` (warm cream)

## Source of truth

The Obsidian note **"Jose Lozano - Full Profile"** is the single source of truth for all personal and professional content on this portfolio. The portfolio is a rendering of that file — not the other way around.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joselozano2003)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/joselozano2003)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
