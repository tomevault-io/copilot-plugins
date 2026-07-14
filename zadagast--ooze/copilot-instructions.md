## ooze-gel-naming

> Call window frame / decorations Ooze Gel, not chrome


# Ooze Gel naming

In docs, UI copy, commit messages, and comments meant for humans:

- Call the app window frame **Ooze Gel** (header bar, traffic lights, drag, resize).
- The continuous pinline cloth across Gel title + OozeKit MAIN BAR / sidebar / status is the **Ooze Gel pinline grid** (`OOZE_PIN_STRIDE`, heights in `aqua-chrome.h`).
- Do **not** say “window chrome”, “Gel chrome”, or “client chrome” in product language.
- OozeKit surfaces/buttons are **OozeKit** finishes / controls — not “Ooze Gel” and not “chrome”.

Internal compositor identifiers for *foreign*-window decorations (e.g. `MyWindowChrome`) may keep the old code names until renamed; new public wording still uses Ooze Gel for Ooze apps.

---
> Source: [Zadagast/ooze](https://github.com/Zadagast/ooze) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-13 -->
