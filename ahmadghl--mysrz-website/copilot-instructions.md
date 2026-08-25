## mysrz-website

> This is the **public site** (`www.mysrztourism.com`). The CMS lives in the separate

# mySRZ Website — agent operating rules

This is the **public site** (`www.mysrztourism.com`). The CMS lives in the separate
`mysrz-admin` repo. Both share one Supabase project and deploy on Vercel.

**Start here:** read `PROJECT-BRIEF.md` (A–Z context) and `SEO-STRATEGY.md` (the living
ranking plan) before any content/SEO work. Every content/SEO change must map to a pillar
and a phase in `SEO-STRATEGY.md` — if it doesn't fit, update the plan first.

## This is not the Next.js you know
Built on Next.js App Router with breaking changes vs stock — APIs, conventions and file
structure may differ from training data. Read the relevant guide in
`node_modules/next/dist/docs/` before writing code, and heed deprecation notices.

## Theme — never hardcode colors
The site uses the warm "Aureate" theme. Colors live as CSS variables in `app/globals.css`
(`--color-primary`, `--color-accent`, `--color-paper`, `--color-on-surface`,
`--color-on-surface-variant`). Use those tokens (or their Tailwind aliases) — do not
hardcode hex/rgb. Long-form content renders through `.prose` / `.markdown-body`; match the
existing typographic styles rather than inventing new ones.

## Content operating rules (non-negotiable — mirrors SEO-STRATEGY.md §2)
- **Research first.** Use SerpApi (`gl=pk`) for keyword intent, People-Also-Ask and the top
  organic URLs, then scrape **at least 10** of those trending sources before writing. Every
  fact/number traceable to a real source.
- **Minimum length: 2,000 words** per post or guide. Thin content does not get indexed on a
  young domain, so the publisher hard-guards this (it refuses drafts under 2,000 words).
- **No em/en dashes** in any published copy. Use commas, colons, or periods. Hard-guard
  the payload before insert.
- **One globally-unique, self-hosted image per surface.** Never reuse an image across
  surfaces; never hotlink. Upload to Supabase `media/content/`, register a `media_files`
  row, and credit the source (the credit *links* to the source page — that link is correct
  and required; the image itself must be self-hosted).
- **Real photos only** for website/blog/destination imagery. AI-generated images are
  social-media only.
- **Publish mechanics:** set BOTH `published:true` AND `status:'published'`; verify the
  page is visible to the anon role; fire `/api/revalidate`; then curl the live URL for a
  200 and the correct `<title>`.
- **SEO/AEO/GEO:** title 30–60 chars, meta description 70–160, keyworded H1/H2s, clean
  slug; 5–6 FAQs as ≤60-word direct answers; dense citable specifics (PKR, km, elevation,
  months, hours); cross-link pillar ↔ destination ↔ related destinations.

## Engineering rules
- `tsc --noEmit` + `npm run build` must be green before any PR.
- **Never push to `main`** — always branch → PR.
- Log meaningful project activity in `ACTIVITY-LOG.md`: context reads, schema or live-data
  assumptions, code/content changes, verification commands, browser checks, deployment notes,
  and any blocker or declined approval.
- Live-DB ops: inspect with a query before and after; never bulk-delete blind.
- Secrets live in `.env.local` / Vercel env (`SUPABASE_SERVICE_ROLE_KEY`,
  `NEXT_PUBLIC_SUPABASE_*`, `REVALIDATE_SECRET`, etc.). Never print or commit secret values.
  Public IDs (AdSense publisher, GA4 measurement) are fine to commit.

---
> Source: [ahmadghl/mysrz-website](https://github.com/ahmadghl/mysrz-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
