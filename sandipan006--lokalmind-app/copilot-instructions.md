## design-system

> Design system rules - tokens, glass surfaces, animations for LokalMind v2.


# Design System Rules

Full reference: `docs/development/CONVENTIONS.md` section 8 · `docs/design/DESIGN-SYSTEM.md`.

## Never hardcode - always use tokens

```typescript
// Colors   → import from '@/src/design-system/tokens/colors'
// Fonts    → import from '@/src/design-system/tokens/typography' (FONTS, FONT_SIZE)
// Spacing  → import from '@/src/design-system/tokens/spacing' (SPACING, BORDER_RADIUS)
```

No hex literals, no named colors (`'white'`), no raw pixel values in StyleSheet.

## Glass surfaces - always use GlassSurface

```typescript
import { GlassSurface } from '@/src/design-system/components';

<GlassSurface tier="heavy">{children}</GlassSurface>           // neutral
<GlassSurface tier="heavy" tint="brand">{children}</GlassSurface> // active/selected
```

Tiers: `ultra` (modals) · `heavy` (cards) · `medium` (input bars) · `light` (highlights)

Never import `GlassView` or `BlurView` in feature code - `GlassSurface` handles fallback.

## Animations

- `react-native-reanimated` only - never `Animated` from `react-native`
- Lottie only for splash and specific branded moments

---
> Source: [Sandipan006/lokalmind-app](https://github.com/Sandipan006/lokalmind-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
