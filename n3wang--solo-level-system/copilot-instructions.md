## solo-level-system

> **Always use `AppColorPalette` - never hardcode colors.**

# Claude Development Guidelines

## Color System

**Always use `AppColorPalette` - never hardcode colors.**

```dart
// ✅ DO
Container(color: AppColorPalette.primary)
Text('Hello', style: TextStyle(color: AppColorPalette.textColor))
Container(color: AppColorPalette.backgroundDarkSurface.withValues(alpha: 0.6))

// ❌ DON'T
Container(color: Colors.blue)
Container(color: Color(0xFF2196F3))
```

**Theme-aware colors:**
- Check `Theme.of(context).brightness == Brightness.dark` for dark mode
- Use `backgroundDarkSurface` for dark mode, `backgroundSurface` for light mode
- Add transparency with `.withValues(alpha: 0.6-0.8)` for backgrounds

## Component Reuse

**Always reuse existing components before creating new ones.**

- `BaseCard` - for card layouts
- `CardHeader` - for card titles/descriptions
- `StatChip` - for key-value displays
- `CustomTextField` - for text inputs
- `PaletteSelectorWidget` - for palette selection
- `ThemeModeSelectorWidget` - for theme selection

Check `lib/widgets/common/` first.

## Code Patterns

**State Management:**
- Use `ValueListenableBuilder` with Hive boxes for reactive updates
- Use `setState` only when necessary

**Theme Detection:**
```dart
final isDark = Theme.of(context).brightness == Brightness.dark;
```

**Palette Colors:**
- `AppColorPalette.primary`, `accent`, `background`, `textColor`
- `AppColorPalette.success`, `error`, `warning`, `info`
- `AppColorPalette.grey50` through `grey900`

## File Organization

- Screens: `lib/screens/`
- Widgets: `lib/widgets/`
- Models: `lib/models/`
- Utils: `lib/utils/`
- Constants: `lib/constants/`

## Quick Commands

- **Add transparency:** `.withValues(alpha: 0.6)`
- **Get palette colors:** `AppColorPalette.getColorsFromPalette(name)`
- **Set active palette:** `AppColorPalette.setActivePalette(name)`
- **Notify palette change:** `PaletteNotifier().notifyPaletteChanged(name)`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/n3wang)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/n3wang)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
