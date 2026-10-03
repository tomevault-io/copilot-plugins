## i18n-and-theme

> next-intl routing, fa default, next-themes


# i18n & dark mode

## i18n (`next-intl`)

- Locales: `en` (LTR), `fa` (RTL). Routes: `/en/...`, `/fa/...`.
- Messages: `messages/en.json`, `messages/fa.json`.
- No hardcoded user-facing strings.
- Full RTL for `fa`: `dir="rtl"`, logical CSS, mirrored layout.
- Locale layout is the main shell: `RootProvider` (`html`/`body` + fonts + `NextIntlClientProvider`) + `AppProviders` (Direction → Query → Theme → Nuqs).
- Root `app/layout.tsx` is pass-through only.
- Fonts: **Inter** (`en`) + **Vazirmatn** (`fa`) via `next/font`, switched by locale.
- `/` resolution: `NEXT_LOCALE` cookie → `Accept-Language` → default **`fa`**.
- `NEXT_LOCALE` written via **next-intl** middleware/helpers.

## Theme (`next-themes`)

- `attribute="class"` (`dark` on `<html>`).
- Default: **system**. User override → browser storage.
- Do not store theme in Zustand.
- Avoid flash: ThemeProvider + `suppressHydrationWarning` when needed.

---
> Source: [soheilhasanjani/contently](https://github.com/soheilhasanjani/contently) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
