## 06-style

> - Always use theming from context when accessing colors, text styles, or other theme values.

## Theming

- Always use theming from context when accessing colors, text styles, or other theme values.
- Use static shorthand methods for theme access:
  - Use `TextTheme.of(context)` instead of `Theme.of(context).textTheme`
  - Use `ColorScheme.of(context)` instead of `Theme.of(context).colorScheme`

## Component Structure

- When generating a component, place the constructor at the top of the class, before fields or methods.
- DON'T use functions like `_buildWidget()` that return widgets. They should be extracted as separate widget instead.
- DON'T include multiple widgets in the same file. Extract them into the component directory under the given feature, or under shared/components if they are reusable.

## Other Style Rules

- Use `const` wherever possible.
- Do not hardcode styles (e.g. colors, padding, typography).
- Avoid deep widget nesting — suggest extracting widgets that exceed 3-4 levels.

---
> Source: [shareefhadid/traak](https://github.com/shareefhadid/traak) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
