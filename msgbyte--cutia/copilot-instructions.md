## i18n

> i18n rules and usage - @i18next-toolkit


# Internationalization (i18n)

The project uses [@i18next-toolkit/nextjs-approuter](https://github.com/moonrailgun/i18next-toolkit/tree/master/packages/react-nextjs-approuter) with `url-segment` routing strategy (`/en/about`, `/zh/about`). All user-facing copy must go through translation.

## Architecture

```
app/
  layout.tsx              ← Root layout: <html>/<body>, minimal
  [locale]/
    layout.tsx            ← Locale layout: I18nProvider + ThemeProvider + generateStaticParams
    page.tsx              ← Home page
    projects/page.tsx     ← Projects page
    editor/[project_id]/  ← Editor page
    ...                   ← All page routes live under [locale]
  api/                    ← API routes (no locale)
  sitemap.ts              ← Multi-locale sitemap with alternates
```

## Setup

- **Config**: `apps/web/src/i18n.config.ts` (createI18nConfig with url-segment strategy)
- **Server Init**: `initServerI18n()` called in `apps/web/src/app/layout.tsx`
- **Root Layout**: `apps/web/src/app/layout.tsx` (minimal: html/body/head)
- **Locale Layout**: `apps/web/src/app/[locale]/layout.tsx` (I18nProvider + providers + generateStaticParams)
- **Provider**: `apps/web/src/components/providers/i18n-provider.tsx` (wraps BaseI18nProvider + bridges i18next for non-React)
- **Navigation**: `apps/web/src/lib/navigation.ts` (locale-aware Link, useRouter, usePathname, redirect)
- **Non-React bridge**: `apps/web/src/lib/i18n.ts` (exports `i18next.t()` for stores/utilities)
- **Middleware**: `apps/web/src/middleware.ts` (createI18nMiddleware for locale detection + URL redirect)
- **CLI Config**: `apps/web/.i18next-toolkitrc.json`
- **Locales**: `apps/web/public/locales/{en,zh,ja,ko,es,pt,fr,de,id,vi,ru,it}/translation.json`

## Usage

### In React components (client)

```typescript
import { useTranslation } from "@i18next-toolkit/nextjs-approuter";

function MyComponent() {
  const { t } = useTranslation();
  return <span>{t("User-facing text")}</span>;
}
```

Interpolation:

```typescript
t("Hello {{name}}", { name: user.name });
```

### In Server Components

```typescript
import { getTranslation } from "@i18next-toolkit/nextjs-approuter/server";

export default async function Page({ params }: { params: Promise<{ locale: string }> }) {
  const { locale } = await params;
  const { t } = await getTranslation(locale);
  return <h1>{t("Hello World")}</h1>;
}
```

### Outside React (stores, utilities, toast)

```typescript
import { i18next } from "@/lib/i18n";

toast.error(i18next.t("Failed to save sound"));
```

Use `i18next.t()` when `useTranslation()` is not available (e.g. Zustand stores, event handlers, utilities).

### Navigation (locale-aware)

Always use locale-aware navigation for internal links:

```typescript
import { Link, useRouter } from "@/lib/navigation";

// Link auto-prefixes href with current locale
<Link href="/projects">Projects</Link>

// useRouter auto-prefixes paths
const router = useRouter();
router.push("/editor/123");
```

**Never** use `next/link` or `next/navigation`'s `useRouter` for internal routes — use `@/lib/navigation` instead.
`next/navigation` is still fine for `useParams`, `notFound`, `useSearchParams`, etc.

### Language switching

```typescript
import { useChangeLocale } from "@i18next-toolkit/nextjs-approuter";

const changeLocale = useChangeLocale();
changeLocale("zh"); // navigates to /zh/current-path
```

### Get current locale

```typescript
import { useLocale } from "@i18next-toolkit/nextjs-approuter";

const locale = useLocale();
```

## Adding New Pages

All new pages must be created under `app/[locale]/`:

```
app/[locale]/my-new-page/page.tsx
```

## Workflow

1. Write copy with `t("...")` or `i18next.t("...")` using the source string as the key.
2. Run `bun run translation:extract` in `apps/web` to extract and merge keys into `public/locales/en/translation.json`.
3. Add translations in each locale file. New locales are initialized with en content; run `translation:translate` or translate manually.

## Rules

- Never hardcode user-facing copy in components or stores.
- Use `t("…")` in React components; use `i18next.t("…")` outside React.
- `t()` / `i18next.t()` must receive a string literal directly (e.g. `t("Popular")`), not a variable/expression (e.g. `t(label)` or `t(labels[key])`), so extraction remains reliable.
- Keep translation keys identical to the default (en) string for extractor compatibility.
- After adding new `t()` / `i18next.t()` calls, run `translation:extract` and update zh translations.
- Do NOT manually create or edit translation JSON files (`public/locales/*/translation.json`). Only write code with `t()` / `i18next.t()` calls; let `translation:extract` handle JSON generation by userself.
- Use `Link` and `useRouter` from `@/lib/navigation`, NOT from `next/link` or `next/navigation`.

---
> Source: [msgbyte/cutia](https://github.com/msgbyte/cutia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
