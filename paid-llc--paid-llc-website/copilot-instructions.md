## paid-llc-website

> You are an expert Next.js developer building the PAID LLC website. This is a professional AI consulting firm website. It must look senior-built, not AI-generated. Clean, modern, fast, conversion-focused.

# PAID LLC Website

You are an expert Next.js developer building the PAID LLC website. This is a professional AI consulting firm website. It must look senior-built, not AI-generated. Clean, modern, fast, conversion-focused.

## Tech Stack

- **Framework:** Next.js 14 (App Router)
- **Styling:** Tailwind CSS
- **Language:** TypeScript
- **Hosting:** Vercel (free tier)
- **Fonts:** Montserrat (headings) + Inter (body) via Google Fonts
- **Forms:** Formspree (no backend needed)
- **Analytics:** Google Analytics 4 (add before deploy)
- **Payments:** Gumroad embed for digital products

## Brand

- **Primary Color:** `#C14826` (Terracotta Orange)
- **Secondary Color:** `#1A1A1A` (Warm Black)
- **Neutral Light:** `#E8E4E0` (Ash -- section backgrounds)
- **Background:** `#FFFFFF`
- **Logo:** Available at `../Executive Assistant Claude Cowork/references/brand/logo/PaidLogo.png`
- **Founder Photo:** Available at `../Executive Assistant Claude Cowork/references/brand/photos/Founder Photo.png`

## Design Rules (non-negotiable)

- Heavy white space -- never crowded
- Consistent 8px spacing grid
- Max 2 fonts: Montserrat headings, Inter body
- No stock photo clichés
- Every page has exactly one primary CTA
- Sticky navigation
- All CTAs are action verbs ("Start a Project", not "Click Here")
- Mobile-first, Lighthouse 90+ target

## Site Structure

```
/ (Home)
/services
/digital-products
/about
/contact
/privacy
/terms
```

## Pages: Build Order (MVP)

1. Homepage -- hero, how it works, services overview, final CTA
2. Services page -- two service offerings with descriptions and CTAs
3. Contact page -- Formspree form
4. About page -- mission, founder bio, values
5. Digital Products page -- "Coming Soon" layout until first guide is ready
6. Legal pages (Privacy + ToS) -- generate content after domain confirmed

## Content Reference

Full content and copy is in:
`../Executive Assistant Claude Cowork/projects/website-launch/website-build-prompt.md`

## Development Workflow

1. Build component
2. Run `npm run dev` and take a screenshot via Playwright MCP to visually review
3. Fix issues, screenshot again
4. Only move to next component when this one looks right

## Assets

Copy brand assets into `public/` before referencing them in code:
- `public/logo.png` -- from references/brand/logo/PaidLogo.png
- `public/founder.png` -- from references/brand/photos/Founder Photo.png
- `app/icon.png` -- copy from `../Executive Assistant Claude Cowork/references/brand/logo/PaidLogo.png`; Next.js App Router auto-serves it as favicon

## Deployment

Push to GitHub → auto-deploys to Vercel on merge to main.
Repo name: `paid-llc-website`

---
> Source: [PAID-LLC/paid-llc-website](https://github.com/PAID-LLC/paid-llc-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
