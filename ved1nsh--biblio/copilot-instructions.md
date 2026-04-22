## biblio

> We use a **MediaQuery scale factor** approach — no `flutter_screenutil`. The design width is **393px**. Sizes stay unchanged on 393px+ screens and smoothly shrink down to 85% on smaller screens.

# Biblio — Claude Code Guidelines

## Responsive UI Strategy

We use a **MediaQuery scale factor** approach — no `flutter_screenutil`. The design width is **393px**. Sizes stay unchanged on 393px+ screens and smoothly shrink down to 85% on smaller screens.
I do not gave git or github initialized to skip that method using them, just generate edits for the files
### The Pattern
```dart
// At the top of your build method:
final screenWidth = MediaQuery.sizeOf(context).width;
final scale = (screenWidth / 393).clamp(0.85, 1.0);

// Then derive sizes from design values:
final titleSize = (28 * scale).clamp(22.0, 28.0);
final iconSize = (44 * scale).roundToDouble();
final padH = (24 * scale).clamp(16.0, 24.0);
```
 
### Rules
- **Scale factor**: Always `(screenWidth / 393).clamp(0.85, 1.0)`. Compute once per `build()`.
- **Font sizes**: `(designSize * scale).clamp(min, max)`. Keep min ~4-6pt below design, max at design value.
- **Container widths/heights**: `(designSize * scale).clamp(min, max)` or `.roundToDouble()`. For aspect-ratio elements (book covers), derive height from width (`coverWidth * 1.45`).
- **Spacing (SizedBox, padding)**: Fine as-is for small values (4-16px). Scale + clamp larger gaps (20+).
- **Border radius**: Fine as-is without scaling.
- **Do NOT scale**: `strokeWidth`, `Duration`, `Curve`, `viewportFraction`, color values, animation multipliers.
- **Rows of equal items** (e.g. day circles): Wrap each item in `Expanded` to prevent overflow, and pass the computed size to the child rather than hardcoding.
- Prefer `IntrinsicHeight` + `Spacer()` over fixed-height cards with stacked `SizedBox` gaps — lets content adapt naturally.
- Prefer `Expanded` + `SingleChildScrollView` over `Column` + `Spacer` for full-screen layouts to avoid overflow.

### Reference
See `lib/instructions.md` for basic sizing table.
See `plan.md` for the phase-by-phase responsive refactor tracker.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ved1nsh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
