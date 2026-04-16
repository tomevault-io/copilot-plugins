## gitsyncmarks

> i18n workflow — add new strings in English first, use key prefixes


# i18n Workflow

- **New user-facing strings:** Add to `_locales/en/messages.json` first.
- **Key prefixes:** `help_` (Help tab), `options_` (options UI), `popup_` (popup), `sync_`, `api_`, etc.
- **Other locales:** Can temporarily miss new keys; runtime falls back to English. Complete translations before release.
- **Placeholders:** Keep `$1`, `$2` in correct positions; move them for grammar in translations.
- **HTML in messages:** Use `data-i18n-html` for trusted content; keep `<code>`, `<a>`, etc. intact in translations.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/d0dg3r)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/d0dg3r)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
