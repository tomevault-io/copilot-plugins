## lightstories

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- `npm run dev` — Start dev server (localhost:3000)
- `npm run build` — Production build
- `npm run lint` / `npm run lint:fix` — ESLint
- `npm run typecheck` — TypeScript type checking (`tsc --noEmit`)
- `npm test` — Playwright E2E tests (requires dev server; auto-starts via webServer config)
- `npm run test:headed` — Playwright in headed mode
- `npx playwright test tests/e2e/homepage.spec.ts` — Run a single test file
- `npx playwright test -g "test name"` — Run a specific test by name

## Architecture

Next.js 16 App Router with `next-intl` for i18n (German default, English). All pages live under `src/app/[locale]/` with locale-based routing using `localePrefix: "as-needed"` (German URLs have no prefix, English uses `/en/`).

### Routing & i18n

- **Routing config**: `src/i18n/routing.ts` — defines locales, exports `Link`, `useRouter`, `usePathname` from `next-intl` (use these instead of Next.js equivalents)
- **Request config**: `src/i18n/request.ts` — loads translation JSON per locale
- **Middleware**: `src/middleware.ts` — `next-intl` middleware for locale detection
- **Translations**: `src/messages/de.json` and `src/messages/en.json`
- **Next.js config**: `next.config.ts` wraps config with `createNextIntlPlugin()`

### Content System

All pages are MDX-driven. Content lives in `src/content/{locale}/` as `.mdx` files, parsed server-side by `src/lib/mdx.ts` using `gray-matter`.

- **Projects**: `src/content/{locale}/projekte/*.mdx` — frontmatter: title, date, category (`hochzeit`|`portrait`|`familie`), heroImage, gallery array, excerpt, optional client/location/testimonial. Supports `draft: true` to hide from production (listings, static generation, direct URL access). Drafts are visible in dev mode with an amber "DRAFT" badge.
- **Category pages**: `src/content/{locale}/{hochzeit,portrait,familie}.mdx` — frontmatter-driven: hero config, gallery config (source: "category" auto-discovers images from `public/images/{category}/`), CTA links. Rendered by shared `CategoryPageRenderer` component
- **Contact page**: `src/content/{locale}/kontakt.mdx` — frontmatter: title, subtitle, contactInfo (email, phone, social links), showContactForm flag. `ContactForm` component still uses `next-intl` translations for form labels
- **Legal pages**: `src/content/{locale}/{impressum,datenschutz}.mdx` — frontmatter: title, metaDescription, layout: "prose". Body content is full markdown rendered via `next-mdx-remote`
- **Page loader**: `getPage(locale, slug)` in `src/lib/mdx.ts` reads any page MDX. `PageFrontmatter` interface covers all page types

### Key Patterns

- **Page params are async**: `params: Promise<{ locale: string }>` — must `await params` before use
- **Static generation**: Pages call `setRequestLocale(locale)` and layouts export `generateStaticParams()` for all locales
- **Components barrel export**: `src/components/index.ts` re-exports all components
- **Contact form**: Client component (`ContactForm.tsx`) with React Hook Form → API route (`src/app/api/contact/route.ts`) → Resend email (`src/lib/email.ts`)
- **MDX body rendering**: `MdxContent` component in `src/lib/markdown.tsx` wraps `next-mdx-remote/rsc` for server-side MDX rendering (used by legal pages)
- **Styling**: Tailwind CSS 4 via PostCSS, global styles in `src/app/globals.css`

### Category Pages

Three photography categories with German route names: `/hochzeit`, `/portrait`, `/familie`. Each reads its MDX file via `getPage()` and delegates to `CategoryPageRenderer`. Gallery images auto-discovered from `public/images/{category}/` via `getCategoryImages()` in `src/lib/images.ts`.

### Troubleshooting

- **Stale images after replacing files**: Next.js caches optimized images in `.next/`. When swapping image files on disk, delete `.next/` and restart the dev server to force a clean rebuild.

## Environment Variables

- `RESEND_API_KEY` — Required for contact form email sending
- `CONTACT_EMAIL` — Recipient email for contact submissions
- `NEXT_PUBLIC_SHOW_PLACEHOLDERS=true` — Display the placeholder tag for images defined in `src/config/placeholder-media.json`

---
> Source: [Travel2Sunrise/LightStories](https://github.com/Travel2Sunrise/LightStories) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
