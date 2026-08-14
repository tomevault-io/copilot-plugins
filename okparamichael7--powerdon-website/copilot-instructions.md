## powerdon-website

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

PowerDon marketing/lead-gen website (Next.js App Router). It's a small set of static-ish marketing pages (home, about, advertising, reserve, contact, legal) with server-action-backed forms that send email via Resend, plus a hand-rolled i18n layer for English/Dutch.

## Commands

- `npm run dev` — start dev server
- `npm run build` — production build (note: TypeScript build errors are ignored — see `next.config.mjs` `typescript.ignoreBuildErrors: true`, so `tsc` is not a build gate)
- `npm run start` — run production build
- `npm run lint` — `next lint`
- `npm run preview-email` — launches `react-email`'s dev server (port 3030) to preview templates in `emails/`

There is no test runner configured in this repo.

Required env vars (not committed; put in `.env.local`): `RESEND_API_KEY`, `FROM_EMAIL`, `TO_EMAIL`, optionally `NEXT_PUBLIC_SITE_URL` (used by `lib/seo.ts`, defaults to `https://powerdon.nl`).

## Architecture

### Dual routing for i18n (the non-obvious part)

Every real page lives **unprefixed** under `app/` (e.g. `app/page.tsx`, `app/contact/page.tsx`, `app/contact/layout.tsx`) and is treated as English/default-locale content. The `app/[locale]/...` tree is a parallel set of **thin re-export shims** for non-default locales, e.g.:

```ts
// app/[locale]/contact/page.tsx
export { default } from "@/app/contact/page";
```

So when adding or editing a page:
- Put the actual component/logic in the unprefixed `app/<route>` file.
- Only touch `app/[locale]/<route>` if a new route needs a shim created (copy the `export { default } from ...` pattern for both `page.tsx` and `layout.tsx`).
- `app/[locale]/layout.tsx` just validates the `locale` param via `isLocale()` and calls `notFound()` for anything unrecognized.

Locale resolution and URL rewriting is centralized in `proxy.ts` (Next's middleware) and `lib/i18n/config.ts`:
- `locales = ["en", "nl"]`, `defaultLocale = "en"`.
- Default-locale URLs are unprefixed (`/contact`); non-default locales are prefixed (`/nl/contact`).
- `proxy.ts` redirects `/en/...` → unprefixed, detects locale from cookie/`Accept-Language` when no prefix is present, sets the `powerdon_locale` cookie, and forwards the resolved locale via the `x-powerdon-locale` request header.
- `lib/i18n/detectLocale.ts` reads that header/cookie server-side (`getRequestLocale`) for use in `generateMetadata`/layouts.
- `lib/i18n/getTranslations.ts` loads the full message tree for a locale; `lib/i18n/useTranslation.ts` (`useTranslation()`, `useLocale()`) is the client-side accessor, sourced from `I18nProvider` (`lib/i18n/provider.tsx`).
- `href(path, locale?)` from `useTranslation()` / `localizePath()` from `lib/i18n/config.ts` should be used for any internal link so it gets the correct locale prefix.

### Translation content is split across two different formats — know which one to edit

- **English** source of truth: `lib/i18n/messages/en.ts` (plain TS object, also the type source — `Messages` type is derived from it) and `lib/i18n/messages/legal-en.ts`.
- **Dutch**: `lib/i18n/messages/nl.ts` doesn't hold copy directly — it imports and assembles JSON files from `locales/nl/*.json` (one file per namespace: `common`, `home`, `about`, `advertising`, `contact`, `reserve`, `forms`, `emails`, `legal`, `seo`) and asserts them against `satisfies Messages`.
- When changing copy that should appear in both languages, update `lib/i18n/messages/en.ts` **and** the matching `locales/nl/*.json` file — they're not auto-synced. See `docs/TRANSLATION_EDITING_GUIDE.md` for the non-technical editing workflow (it explains which JSON file maps to which page, and warns against touching `{placeholder}` tokens).

### Forms → validation → email pipeline

Each lead-gen form (contact, reserve/partnership, advertising/campaign quote, pilot testing) follows the same three-layer flow:

1. **Schema** (`schema/schema.ts`): a `create*Schema(copy)` factory per form that takes localized validation-message copy and returns a Zod schema (so error messages are localized). Static `en`-based schemas are also exported for client-side use.
2. **Server action** (`app/actions/{contact,reserve,campaign,pilot}.ts`, all `"use server"`): re-validates with the locale-aware schema, then calls into `lib/mail.ts`, and returns `{ success | error, status }` using localized strings from `getTranslations(locale)`.
3. **Email sending** (`lib/mail.ts`): uses Resend (`RESEND_API_KEY`) to send an admin notification (React Email template from `emails/`) and a localized confirmation email to the submitter. Partnership/advertising/pilot emails are tagged with a generated request ID (`generateRequestId("PTN"|"CAM"|"PLT")`) that's included in both the admin and confirmation subject lines.

When adding a new form, mirror this pattern rather than inlining fetch/validation logic in the component.

### SEO metadata

- `lib/seo.ts` holds `siteConfig` (org info, contact details, social links) and `generateMetadata()`, the single function that builds Next `Metadata` (OpenGraph, Twitter card, canonical + per-locale `alternates.languages`) for every page.
- `lib/seo-page.ts`'s `getLocalizedPageMetadata(pageKey)` is the shortcut most page `layout.tsx` files call from `generateMetadata()` — it resolves the request locale, pulls `messages.seo.pages[pageKey]`, and maps the page key to its canonical path via the internal `pagePaths` table. If you add a new top-level route that needs SEO metadata, add its key to `Messages["seo"]["pages"]` (in `en.ts`/`nl` JSON) and to `pagePaths` in `lib/seo-page.ts`.
- `schema/index.tsx` + `components/seo/` render JSON-LD structured data server-side (`StructuredDataServer`), using the schema builders in `lib/seo.ts` (`getOrganizationSchema`, `getLocalBusinessSchema`, etc.).

### UI components

`components/ui/` is a standard shadcn/ui setup (`components.json`, Radix primitives, `class-variance-authority`, Tailwind). Use the existing `cn()` helper (`lib/utils.ts`) for conditional classNames and prefer composing existing `components/ui/*` primitives over new bespoke ones.

---
> Source: [okparamichael7/powerdon-website](https://github.com/okparamichael7/powerdon-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
