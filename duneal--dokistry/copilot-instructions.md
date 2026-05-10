## scss-variables-mixins

> Use predefined SCSS variables and mixins instead of hardcoded values. Always import the base SCSS properly.


- Use SCSS variables like `$spacing-`, `$c-primary`, `$c-secondary`, `$fs-`, `$fw-`, `$bkp-`, etc., defined in `src/styles/base/_variables.scss`.
- Use mixins such as `@include flex()`, `@include grid()`, `@include title()`, `@include text()`, etc., defined in `src/styles/base/_mixins.scss`.
- Do not hardcode values (e.g., `px`, `#fff`) directly in SCSS, unless in specific justified cases.
- Always import the base SCSS with `@use '../base' as *;`, adjusting the relative path as needed.

---
> Source: [duneal/dokistry](https://github.com/duneal/dokistry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
