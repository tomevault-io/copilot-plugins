## angular-ngx-translate

> Angular UI copy uses ngx-translate; add keys to all locale JSON files


# Angular – Translations (`ngx-translate`)

- **User-visible strings** in templates should use the **`translate` pipe** (or `TranslateService` in TS), not raw literals — match surrounding components (e.g. `{{ 'SETTINGS.TITLE' | translate }}`).
- **Keys:** Use the existing naming style (`SECTION.SUBKEY`). Prefer reusing **`COMMON.*`** keys for shared actions (Save, Cancel, Delete) when they already exist.
- **Locale files:** Add every new key to **`front/public/i18n/*.json`** for all shipped languages (keep objects in sync across `en.json`, `de.json`, `es.json`, etc.). Do not add keys to only one file unless the task explicitly allows a partial rollout.
- **Placeholders / attributes:** Use `[placeholder]="'KEY' | translate"` or `translate` directive patterns already used in the codebase.
- Deeper context: **`AGENTS.md`** (smoke tests), **`docs/testing.md`** for Puppeteer scripts after UI changes.

---
> Source: [satisfecho/pos](https://github.com/satisfecho/pos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
