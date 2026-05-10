## i18n-guidelines

> i18n usage with WXT and consistency checks

# i18n Guidelines

- Use the helper in [`src/utils/i18n.ts`](mdc:src/utils/i18n.ts): `t(id, substitutions?)`
  - Delegates to WXT i18n when available, falls back to `browser.i18n`
  - Supports string, array, and number substitutions
- Store translation keys in [`src/locales/*.json`](mdc:src/locales)
- Consider English (`en.json`) the base; keep keys in other locales aligned
- Validate locale coverage with:
```bash
pnpm run check:i18n
```
- Avoid hardcoded user-facing strings; prefer `t('key.path')`

---
> Source: [RonkTsang/gemini-chat-extension](https://github.com/RonkTsang/gemini-chat-extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
