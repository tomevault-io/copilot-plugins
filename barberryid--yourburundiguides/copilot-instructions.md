## yourburundiguides

> - **Site:** yourburundiguides.com (production `https://www.yourburundiguides.com`)

# CLAUDE.md — Your Burundi Guides

## Project Reference

- **Site:** yourburundiguides.com (production `https://www.yourburundiguides.com`)
- **Type:** Astro static marketing site for a private local guiding service in Bujumbura, Burundi (lead guide Gabriel Ndikumana). It is **WhatsApp-first** — the primary conversion action is contacting the guide on WhatsApp.
- **Stack:** Astro 5 (static output, `trailingSlash: 'always'`), Tailwind CSS v4 via `@tailwindcss/vite`, `@astrojs/sitemap`, GitHub, Cloudflare Pages.
- **GitHub:** https://github.com/barberryid/yourburundiguides
- **Cloudflare preview:** https://yourburundiguides.pages.dev/

### Paths

- Project folder: `C:\Users\Gary\code\yourburundiguides`
- Optimised site media: `public/images/<section>/` (hero, guides, gallery, reviews, og)
- Raw/working media: kept out of git — `.gitignore` covers `contact-sheets` and `whatsapp images 12062026`; retired working assets (source docx, scraped reference images, contact sheets, raw WhatsApp images) live in `archive/`

### Build (Git Bash)

```bash
cd "/c/Users/Gary/code/yourburundiguides"
npm run build
```

PowerShell: `cd "C:\Users\Gary\code\yourburundiguides"`

### Publish changes

```bash
git status
git add .
git commit -m "Update Your Burundi Guides"
git push
```

Cloudflare Pages auto-deploys the pushed commit from `main`.

## Conventions

- **Content lives in `src/data/*.ts`.** Edit `site.ts` (global metadata, WhatsApp link, email, guide profiles, per-page titles/descriptions), plus `services.ts`, `prices.ts`, `reviews.ts`, `gallery.ts`, and `faqs.ts`. Update these data files rather than hard-coding repeated content in page templates.
- **Pages are fixed `.astro` files** in `src/pages/` (no content collection). A new page is a new file, served at `/file-name/` (note the trailing slash).
- **Styling is Tailwind CSS v4, CSS-first.** Global tokens and component classes live in `src/styles/global.css` (`@import "tailwindcss";` then an `@theme` block). Do **not** create a `tailwind.config.js` — v4 is configured in CSS. Design direction: warm cream background, deep green headings/nav, light cards, WhatsApp green reserved for WhatsApp actions, gold accent for focus/emphasis.
- **WhatsApp:** the number and `whatsappLink` are centralised in `src/data/site.ts`. Keep all WhatsApp CTAs pointed at that central value.
- **Media:** only add optimised copies to `public/images/`. Keep raw originals out of git.
- Full setup, structure, homepage flow, SEO, and deployment docs are in `README.md`.

---
> Source: [barberryid/yourburundiguides](https://github.com/barberryid/yourburundiguides) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
