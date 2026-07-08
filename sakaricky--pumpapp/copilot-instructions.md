## ui-and-intl

> UI work must use i18n (intl); add or update translations for any user-visible text


# UI and internationalization (intl)

- **Always use the project’s i18n** for user-visible strings. Use `useTranslation()` from `react-i18next` and render text via `t('key')` (or `t('namespace.key')`). Do not hardcode English or French (or any locale) in UI components.

- **When adding or changing UI copy**, add or update the corresponding keys in both locale files:
  - `apps/web/src/locales/en.json`
  - `apps/web/src/locales/fr.json`  
  Keep keys in sync and provide appropriate English and French text. If a translation is uncertain, add a sensible placeholder and mark it (e.g. with a comment in the JSON or a TODO) so it can be refined later.

- **Scope:** Buttons, labels, titles, placeholders, error messages, aria-labels, and any other text the user sees. Non-UI code (e.g. API, shared types) does not use these locale files.

- **Structure:** Reuse the existing key structure (e.g. `nav.*`, `home.*`, `shifts.*`, `reports.*`) and add new namespaces or keys as needed for new features.

---
> Source: [SakaRicky/PumpApp](https://github.com/SakaRicky/PumpApp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
