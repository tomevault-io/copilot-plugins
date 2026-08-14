## dinopizza-nextjs

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # Start dev server with Turbopack
npm run build    # Production build
npm run start    # Start production server
npm run lint     # Run ESLint
```

## Architecture

**DinoPizza** is a restaurant website for a Greek pizzeria, built with Next.js 15 App Router, React 19, and `next-intl` for bilingual (Greek/English) support.

### Routing & Internationalization

All pages live under `src/app/[locale]/`. The `next-intl` middleware (configured in `src/middleware.js`) intercepts every request and prepends the locale. Greek (`el`) is the default locale; English (`en`) is the alternative.

- Locale routing config: `src/i18n/routing.js`
- Translations: `messages/en.json` and `messages/el.json`
- Navigation helpers (locale-aware `<Link>`, `useRouter`): `src/i18n/navigation.js`

### Data Layer

All product/category content is stored in `data/products.json` — a single bilingual JSON file with nested categories, subcategories, products, sizes, and prices.

Helper functions in `lib/dataHelpers.js` handle:
- Slug-based lookups (`findCategoryBySlug`, `findProductBySlug`)
- Slug translation between locales (`getSlugInLocale`, `getLocalizedCategorySlug`, `getLocalizedProductSlug`)

When adding or editing menu items, `data/products.json` is the only file to update. Both `en` and `el` slug/name/description fields must be populated.

### Dynamic Menu Routes

```
/[locale]/menu/[category]/[product]
```

These pages use `generateStaticParams` to pre-render all category and product combinations from `products.json`. If slugs change in the JSON, static params must stay in sync.

### Server Actions & Email

Form submissions use Next.js Server Actions in `src/actions/`:
- `SendContactEmail.js` — contact form
- `SendSubscriptionEmail.js` — newsletter subscription

Both use nodemailer via `src/api/config.js` with Hostinger SMTP (requires `SMTP_USER` and `SMTP_PASS` env vars).

### Custom Pizza Builder

`/[locale]/yourpizza` — pricing logic lives in `commons/utils.js` (`calculatePizza`). Available toppings are defined in `commons/contants.js`.

### Styling

The project uses **legacy CSS** (Bootstrap + jQuery plugins), not Tailwind. Stylesheets are in `public/css/`, and jQuery/plugin scripts are in `public/js/`. These are loaded globally via the root layout.

### SEO & Analytics

- Per-page metadata generated via `lib/generateMetadata.js`
- Google Analytics via `@next/third-parties/google` (requires `NEXT_PUBLIC_GOOGLE_ANALYTIC_CODE` env var)
- Cookie consent gates analytics tracking — `<GoogleAnalytics>` is only rendered after consent is granted, inside `components/CookieConsent/CookieConsentWrapper.js`
- Vercel Analytics and Speed Insights included in the root layout

## Environment Variables

```
SMTP_USER            # Hostinger SMTP username
SMTP_PASS            # Hostinger SMTP password
NEXT_PUBLIC_GOOGLE_ANALYTIC_CODE # GA4 measurement ID (must be NEXT_PUBLIC_ since it's read client-side to gate on cookie consent)
```

---
> Source: [Spanosnick/dinopizza-nextjs](https://github.com/Spanosnick/dinopizza-nextjs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
