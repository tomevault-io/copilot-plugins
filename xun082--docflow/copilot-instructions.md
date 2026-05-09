## 11-global-components-utils

> Global components and utilities conventions


When working in `components/**` and `utils/**`:

- Components under `components/**` are reusable across the app; keep them pure and well-typed.
- `utils/**` contains side-effect-free helpers, constants, and data transforms.
- Naming: PascalCase for components/types/enums; camelCase for variables/functions/hooks; kebab-case for file names.
- Avoid framework-specific assumptions in `utils/`.
- Provide examples and minimal docs in code comments where helpful (self-explanatory first).

---
> Source: [xun082/DocFlow](https://github.com/xun082/DocFlow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
