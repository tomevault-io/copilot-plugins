## dotfiles

> Enforce Biome as the global linter/formatter and discourage use of ESLint/Prettier unless justified in README.


**Prompt for Biome Usage**

Your changes must align with the tooling described in the `🎯 Philosophy` section of `@README.md`.

✅ Use `biome` as the default and only global linting/formatting tool for node:

- `biome.json` should be present and configured
- Do not introduce `.eslintrc`, `.prettierrc`, or related config files unless justified and documented in the README

🛑 Do not install or configure ESLint or Prettier globally.

📝 If you're modifying or suggesting linting rules, ensure they target Biome and reflect the intended setup outlined in the README.

**Context File:**  
@README.md

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/agwhi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
