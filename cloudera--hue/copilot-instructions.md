## hue

> SCSS + BEM rules; forbid hardcoded colors; attach to style files.


# Styling Rules
- **Primary:** SCSS with BEM; avoid adding new Less. Do not automigrate when touching unless you can easily migrate the entire file.
- **No hardcoded colors**; use SCSS variables/tokens (cuix/antd) from 
- Co-locate `ComponentName.scss` with the component; keep selectors BEM-scoped.
- Prefer BEM modifiers over state classes
- Prefer classes over using HTML element selectors

## Example
```scss
@use 'variables' as vars;
.component-name {
  &__title {  color: vars.$hue-primary-color-dark;; }
  &--disabled { opacity: .5; cursor: not-allowed; }
}
```

---
> Source: [cloudera/hue](https://github.com/cloudera/hue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
