## portfolio-nextjs

> SEO and metadata practices for Next.js App Router with i18n


# SEO

## Metadata

- Default site metadata and OG/Twitter cards are defined in [src/constants/metadata.ts](mdc:src/constants/metadata.ts).
- Export `metadata` in routes/layouts when page-specific metadata is needed. Ensure canonical and `alternates.languages` are set for localized pages.
- Inject JSON-LD schema with [src/components/common/JsonLdSchema/index.tsx](mdc:src/components/common/JsonLdSchema/index.tsx) as done in the locale layout.

## Localization

- Keep `html lang` correct via [src/app/[locale]/layout.tsx](mdc:src/app/%5Blocale%5D/layout.tsx).
- Provide localized content via `next-intl`; avoid hard-coded strings in components. Use [src/hooks/useTranslation.ts](mdc:src/hooks/useTranslation.ts) and messages in `public/locales/*`.

## Content semantics

- Use one `<h1>` per page. Keep headings hierarchical.
- Provide descriptive `alt` for images and meaningful link texts.

## Social

- OG and Twitter images live under `public/images/seo`. Reference them via the constants in [src/constants/metadata.ts](mdc:src/constants/metadata.ts).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yoelvyspc93) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
