## fivfold-site

> FivFold docs site rules — Next.js, Tailwind v4, docs conventions


# FivFold Docs Site

When modifying the site package:

- **Tech stack:** Next.js 14+ (App Router), React 18+, Tailwind v4, shadcn
- **Styling:** Tailwind v4 only; use `@import "tailwindcss"`
- **Conventions:** Follow Next.js and React best practices; reference the React best-practices skill when optimizing performance or data patterns

## SEO Requirements

When adding or modifying pages in the site:

1. **New page:** Export `metadata` (or `generateMetadata`) with:
   - `title` — unique, descriptive
   - `description` — 120–160 chars, accurate to page content
   - `openGraph` and `twitter` overrides when the page has distinct content

2. **Client component pages:** If the page uses `"use client"`, add a `layout.tsx` in the same route segment that exports metadata. The layout wraps the client page.

3. **Sitemap:** Add the new route to `site/app/sitemap.ts` with appropriate `priority` and `changeFrequency`.

4. **When updating page content:** Update the page's `metadata` (title, description) to reflect the changes.

---
> Source: [Fivex-Labs/fivfold](https://github.com/Fivex-Labs/fivfold) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
