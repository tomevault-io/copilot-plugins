## ooze-kit-typography

> OozeKit Inter 11pt regular typography


# OozeKit typography

Default UI type for OozeKit apps and shell chrome:

- **Family:** Inter
- **Size:** 11pt
- **Weight:** 400 (regular)

Do not use medium/semibold/bold in kit or app chrome CSS unless the product explicitly needs emphasis. Opt in with the `.ooze-emphasis` class (weight 600) or `OOZE_UI_FONT_EMPHASIS` for Pango.

Call `ooze_theme_ensure()` once at app startup. Font tokens live in `common/ooze-font.h`.

Terminal text (Ooze Command VTE) stays monospace — only chrome uses Inter.

---
> Source: [Zadagast/ooze](https://github.com/Zadagast/ooze) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-13 -->
