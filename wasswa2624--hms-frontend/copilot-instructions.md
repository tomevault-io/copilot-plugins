## hms-frontend

> Define the visual system (tokens, themes, styling constraints) for all platforms.


# Theme & Design System Standards

## Purpose
Define the visual system (tokens, themes, styling constraints) for all platforms.
This file owns styling policy and design token usage.

## Theme Ownership
Theme layer owns:
- Color, spacing, typography, radius, shadow tokens
- Breakpoints and motion tokens
- Theme resolution and shape

Theme layer does not own:
- UI rendering logic
- Domain business logic

## Folder Structure
```txt
src/theme/
  index.js
  breakpoints.js
  animations.js
  tokens/
    colors.js
    spacing.js
    typography.js
    radius.js
    shadows.js
  light.theme.js
  dark.theme.js
```

## Theme Variants
- Only two themes are supported: `light` and `dark`.
- Both themes must share identical object shape.
- Theme switching is controlled outside UI components (store/bootstrap).

## Token Rules
- Tokens are semantic and platform-agnostic.
- No hardcoded visual values in UI files.
- Hex values belong only in token files.

## Styling Rules
- Styled-components is the default styling approach.
- Styled components are defined in `*.styles.jsx` files.
- Inline styles are forbidden for production UI.

### Native `StyleSheet` Exception
`StyleSheet` is allowed only in:
- `*.android.styles.jsx`
- `*.ios.styles.jsx`

And only for static, token-driven, performance-sensitive styles.

### Styled-Components Entrypoints
- `*.android.styles.jsx` -> `styled-components/native`
- `*.ios.styles.jsx` -> `styled-components/native`
- `*.web.styles.jsx` -> `styled-components`

## Platform Styling Strategy
- Shared style file first (`Component.styles.jsx`).
- Platform override style files only when required.
- No `Platform.OS` branching in style files.
- Override files must preserve the same theme API shape.

## Accessibility and Design Direction
- Contrast and focus visibility must satisfy accessibility requirements.
- Motion must respect reduced-motion preferences.
- Visual direction follows Microsoft Fluent / Microsoft 365 style principles.

## Related Owners
- UI file structure: `component-structure.mdc`
- Provider placement and theme init: `bootstrap-config.mdc`
- Accessibility contrast and motion criteria: `accessibility.mdc`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/WASSWA2624) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
