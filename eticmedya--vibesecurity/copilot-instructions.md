## vibesecurity

> Vibe Security — security audit library.

Vibe Security — security audit library.

This repo is a bilingual (TR/EN) security-audit checklist for AI-built web and mobile apps.

When the user asks to audit / check security / "denetle":
- Follow prompts/START-HERE.md: FIRST ask whether it's a web or a mobile app.
- Checks live in checks/web/ (1-30) and checks/mobile/ (M1-M10). Index: checks/checks.json.
- GOLDEN RULE: every check must FIND and REPORT first, and only FIX after the user approves.
  Never make blind changes.
- Work in the list's order: Medium -> High -> Critical (warm up first, most critical last). If the user is in a hurry, start with Critical.
- After each fix, remind the user of that check's "Verify" step.
- If unsure, say "not sure" — do not guess.

This is a defensive tool, not a substitute for a penetration test.
Author: Aykut Üçeş — x.com/aykutuces

---
> Source: [eticmedya/Vibesecurity](https://github.com/eticmedya/Vibesecurity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
