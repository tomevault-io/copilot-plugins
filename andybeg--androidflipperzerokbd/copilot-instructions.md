## no-cursor-commit-trailer

> Never add Cursor co-author trailers to git commits


# Git commits — no Cursor agent attribution

Do **not** put Cursor / agent attribution in commit messages.

- Never add `Co-authored-by: Cursor <cursoragent@cursor.com>` (or similar).
- Credit for Cursor help already lives in `README.md` (Credits section).
- If a commit hook or the environment injects that trailer, strip it before push (rewrite the message / amend only when amend rules allow, otherwise filter-branch / new commit as appropriate).

---
> Source: [andybeg/AndroidFlipperZeroKBD](https://github.com/andybeg/AndroidFlipperZeroKBD) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
