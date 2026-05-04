## frontend-i18n

> Frontend internationalization requirements for web UI changes


# Frontend Internationalization

- Any user-facing copy in `web/src` must be localizable.
- Add new strings to `web/src/i18n.ts` in both `en` and `zh`.
- Do not hardcode labels, buttons, tooltips, empty states, errors, status text, or helper text directly inside components.
- Use `t()` for UI strings and localized objects like `{ en, zh }` for catalog-style content.
- When adding a feature, translate all supporting states too: loading, success, failure, empty, confirmation, logs, and hints.
- Keep translation keys stable and grouped by feature prefix.

```tsx
// ❌ BAD
<button>Install</button>

// ✅ GOOD
<button>{t("aiApps.install")}</button>
```

---
> Source: [OpenCSGs/csghub-lite](https://github.com/OpenCSGs/csghub-lite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
