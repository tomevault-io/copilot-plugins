## text-selectability

> Text selectability — native desktop UX for text selection and cursor behavior


# Text selectability

`<body>` has `user-select: none` and `cursor: default` globally so UI chrome behaves like a native desktop app.

Add `select-text cursor-text` to elements showing **user data the user might want to copy** (filenames, paths, version strings, error messages, URLs, etc.). Leave UI chrome (headers, labels, buttons, tags, badges) non-selectable.

Inside clickable cards/rows, prefer making data selectable in the detail panel rather than on the click target itself to avoid click-vs-drag friction.

---
> Source: [cyberpunk2073/vam-backstage](https://github.com/cyberpunk2073/vam-backstage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
