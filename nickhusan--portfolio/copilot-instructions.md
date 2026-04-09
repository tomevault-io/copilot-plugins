## portfolio

> - Next.js 15 App Router + React 19

# Portfolio — nick.dev

## Stack
- Next.js 15 App Router + React 19
- Tailwind CSS 4 (no config file, uses CSS vars in globals.css)
- Framer Motion 12 — scroll animations (`AnimateIn` component)
- Lenis — smooth scroll (initialized in `SmoothScroll` client component)
- gray-matter — markdown frontmatter parsing
- next-mdx-remote/rsc — MDX rendering in Server Components
- @vercel/analytics — analytics (zero-config on Vercel)

## Content
All content lives in `content/` as `.md` files. Parsed by `src/lib/content.ts`.
Push a new `.md` file → Vercel auto-deploys it.

### Schemas
- `content/learn-log/*.md` — `title, date, tags[], summary`
- `content/llm-reviews/*.md` — `title, model_name, parameter_count, quantization, vram_required_gb, gpu_setup, tensor_parallel, context_length, tokens_per_sec, time_to_first_token_ms, rating, pros[], cons[], date`
- `content/papers/*.md` — `title, authors[], venue, year, abstract, pdf_url?, code_url?`
- `content/projects/*.md` — `title, description, tags[], url?, github_url?, date, content` (content is MDX body)
- `content/blog/*.md` — `title, date, tags[], summary` (content is MDX body)

## Key files
- `src/config/site.ts` — name, social links, nav items (edit this first)
- `src/lib/types.ts` — TypeScript interfaces
- `src/lib/content.ts` — all markdown parsing (getAll*, get* by slug)
- `src/lib/utils.ts` — reading time utility
- `src/lib/terminal-commands.ts` — terminal command registry + easter eggs + pipe support
- `src/components/ui/AnimateIn.tsx` — scroll-triggered fade-in wrapper (respects `prefers-reduced-motion`)
- `src/components/ui/BackToTop.tsx` — scroll-to-top button
- `src/components/ui/CommandPalette.tsx` — Cmd+K search across all content
- `src/components/ui/CustomCursor.tsx` — themed cursor ring (pointer-fine only)
- `src/components/mdx/` — MDX component overrides (Pre with copy button)
- `src/components/home/ExperienceSection.tsx` — timeline (edit inline data here)
- `src/components/home/ReferencesSection.tsx` — references/testimonials (edit inline data here)

## Routes
- `/` — Homepage (Hero, About, Experience, Projects, Papers, References, Contact)
- `/blog` — blog listing with tag filter
- `/blog/[slug]` — blog post detail with MDX
- `/papers` — dedicated papers listing page
- `/resume` — structured resume with download PDF button
- `/research` — research interests with status indicators
- `/now` — /now page (what I'm currently working on)
- `/learn-log` — listing with tag filter
- `/learn-log/[slug]` — entry detail with MDX
- `/llm-reviews` — listing with sort filter
- `/llm-reviews/[slug]` — review detail with MDX
- `/projects/[slug]` — project detail with MDX
- `/terminal` — interactive terminal
- `/feed.xml` — RSS feed (learn log)
- `/sitemap.xml` — auto-generated sitemap
- `/api/search` — search data JSON (used by command palette)

## Conventions
- Server Components by default; add `"use client"` only when needed
- Dark theme only — no light mode toggle
- `prose-invert` classes for MDX content styling
- Font vars: `--font-geist-sans`, `--font-geist-mono`
- MDX pages pass `components={mdxComponents}` for copy button on code blocks
- OG images auto-generated per route via `opengraph-image.tsx` files

## Accessibility
- `prefers-reduced-motion` disables all animations (AnimateIn, HeroSection, Lenis)
- `focus-visible` outlines for keyboard navigation (globals.css)
- `aria-label` on nav, `aria-current="page"` on active nav link
- Custom cursor only on `(pointer: fine)` devices

## Terminal
Commands: `help`, `ls`, `cat <file>`, `whoami`, `pwd`, `neofetch`, `nvidia-smi`, `history`, `cowsay`, `matrix`, `echo`, `sudo`
Files: `about.md`, `blog.md`, `contact.md`, `projects.md`, `research.md`, `README.md`
Pipes: `cmd | grep <pattern>`, `cmd | head -n <N>`, `cmd | wc -l`

## To add content
```bash
# New learn log entry
touch content/learn-log/my-entry.md

# New LLM review
touch content/llm-reviews/model-name.md

# New project (add MDX body content for detail page)
touch content/projects/project-name.md

# New blog post
touch content/blog/post-slug.md
```

## Deploy
Push to main → Vercel auto-deploys. No build step needed locally unless checking types.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nickhusan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nickhusan)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
