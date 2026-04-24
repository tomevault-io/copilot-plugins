## calico-containerlab-site

> A Next.js site that serves as an interactive learning hub for Kubernetes networking with Calico, using ContainerLab for hands-on lab exercises.

# Calico ContainerLab Site

A Next.js site that serves as an interactive learning hub for Kubernetes networking with Calico, using ContainerLab for hands-on lab exercises.

## Stack

- **Next.js 16** (App Router) + **React 19** + **TypeScript**
- **Tailwind CSS v4** (PostCSS-based, not config-based — no `tailwind.config.js`)
- **react-markdown** with `remark-gfm`, `rehype-raw`, `rehype-slug` for markdown rendering
- **mermaid** for diagram rendering (lazy-loaded client-side)

## Dev Commands

```bash
npm run dev      # start dev server
npm run build    # production build
npm run lint     # eslint
```

## Architecture

Content is fetched remotely — there are no local markdown files. All lesson content lives in a separate GitHub repo:
- **Content repo**: `github.com/aadhilam/k8-networking-calico-containerlab` (branch: `master`)
- **Content path**: `containerlab/<slug>/README.md` (tries `README.md`, `Readme.md`, `readme.md`)
- Fetched at request time with `revalidate: 3600` (1-hour ISR cache)

### Key Files

| File | Purpose |
|------|---------|
| `src/lib/lessons.ts` | All lesson data, fetch logic, URL rewriting, heading extraction |
| `src/app/page.tsx` | Home page — lesson card grid |
| `src/app/lessons/[slug]/page.tsx` | Dynamic lesson page — video embed, markdown, TOC, related lessons |
| `src/app/repo-setup/page.tsx` | Repo setup page — fetches root `readme.md` |
| `src/app/lab-setup/page.tsx` | Lab setup page — fetches `containerlab/readme.md` |
| `src/components/MarkdownRenderer.tsx` | Client component — renders markdown with callouts + mermaid |
| `src/components/LessonCard.tsx` | Card used on home page grid |
| `src/components/LessonScroller.tsx` | Horizontal scroller used in "Explore more" section |
| `src/components/TableOfContents.tsx` | TOC sidebar (headings h1–h4) |
| `src/components/ProgressTracker.tsx` | Tracks completed lessons (likely localStorage) |
| `src/components/ThemeToggle.tsx` | Dark/light toggle — persists to `localStorage`, sets `data-theme` on `<html>` |

### Adding a New Lesson

1. Add an entry to `LESSON_META` in `src/lib/lessons.ts` with slug `NN-name`, title, image path, and optional `youtubeId`
2. Slug format: zero-padded number prefix e.g. `21-new-topic`
3. Add a thumbnail image to `public/images/`
4. The content README must exist in the content repo at `containerlab/<slug>/README.md`

### URL Rewriting (`rewriteUrls`)

`fetchLessonContent` rewrites markdown before rendering:
- Relative image paths → absolute `raw.githubusercontent.com` URLs
- `../readme.md` links → `/lab-setup` internal links

### Callout Syntax

`MarkdownRenderer` supports GitHub-style callouts inside blockquotes:
```md
> [!NOTE] This is a note
> [!WARNING] This is a warning
> [!TIP], [!IMPORTANT], [!CAUTION] also supported
```

### Theme

- Default: dark. Theme stored in `localStorage` key `theme`.
- CSS variables (`--accent`, `--bg-secondary`, `--border`, `--text-primary`, `--text-secondary`, `--bg-card`, `--bg-card-hover`) defined in `globals.css` and toggled via `data-theme` attribute on `<html>`.
- Mermaid diagrams re-render on theme change via `MutationObserver`.

## Constraints & Notes

- **No local content files** — don't create markdown lesson files locally; all content is in the external repo
- **Static params** generated from `LESSON_META` at build time via `generateStaticParams`
- Lesson `01-calico-ipam` uses a different font (`Noto_Sans_Math` + `JetBrains_Mono`) applied via a `fontVars` wrapper div — other lessons use the global Geist fonts
- The `@/` path alias maps to `src/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aadhilam) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
