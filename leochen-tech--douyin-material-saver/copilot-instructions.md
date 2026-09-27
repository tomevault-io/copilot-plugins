## douyin-material-saver

> - Do not use `adb shell input text` for Douyin URLs or other punctuation-sensitive text. It can silently drop `:`, `/`, `.`, and other characters.

# Project testing notes

## Android device text input

- Do not use `adb shell input text` for Douyin URLs or other punctuation-sensitive text. It can silently drop `:`, `/`, `.`, and other characters.
- Inject test share text with an `ACTION_SEND` intent (or a clipboard API when that flow is specifically under test), then read the UI hierarchy and confirm the complete original URL is present before starting the parse.
- During parse and preview checks, do not tap the save-to-gallery action unless the user explicitly asks for a real save test.

---
> Source: [LeoChen-Tech/douyin-material-saver](https://github.com/LeoChen-Tech/douyin-material-saver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
