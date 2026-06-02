## accessibility

> Accessibility standards for OscarWatch UI (vision, color blindness, contrast)


# OscarWatch accessibility

Target **WCAG 2.1 Level AA** for text and essential UI. Custom map/sky plots must remain usable without relying on color alone.

## Color and color blindness

- **Never encode meaning with hue only.** Satellite/track/footprint identity must always have a **text label** (map) or be identifiable in lists/dialogs by name. Future: optional dash patterns or marker shapes per index.
- **Satellite palette:** keep the **Okabe–Ito set** in `PlotColors.cs`; avoid pairs that collide for deuteranopia/protanopia (red vs green, cyan vs green, yellow vs lime).
- **Status (pass imminence, stale TLE, errors):** pair color with **words** (“AOS in …”, “Stale TLE”). Use `PassHighlightBrush` / `StaleTleForegroundBrush` from `AccessibilityThemeResources`, not ad-hoc hex.
- **Ground station / satellites:** use `PlotMarkerDrawing` (dark halo + white ring); dashed ring for below-minimum elevation on sky plot.

## Contrast and theme

- **Body text:** ≥ **4.5:1** against background (light and dark theme). **Large text (≥18pt or 14pt bold):** ≥ **3:1**.
- **Chart chrome** (rings, spokes, grid lines): ≥ **3:1** against plot background in `UiPalette` for both themes.
- **Plot overlays** (tracks, footprints): stroke ≥ **3:1** vs average map/plot background under that region; increase width or alpha before picking a louder hue.
- Prefer `{DynamicResource SystemControlForeground*}` / `UiPalette` over hardcoded hex in XAML. New brushes go in `App.axaml` or `UiPalette.cs` with **both** light and dark values checked.
- **Do not use opacity below 0.85** for text required to understand passes or telemetry; secondary hints may use `BaseMedium` brush, not faded `Opacity="0.72"` on body copy.

## Typography and layout

- **Minimum 12px** for readable content (pass rows, settings labels, live telemetry). **10px only** for non-essential hints; never for times, coordinates, or az/el alone.
- Support **system text scaling** where possible; avoid fixed tiny heights that clip text when OS scaling &gt; 100%.
- **Monospace** for times/coordinates is fine; keep digit width consistent (`Consolas`, `Cascadia Mono`, etc.).

## Keyboard and focus

- All interactive surfaces: `Focusable`, visible **focus indicator** (Fluent theme), logical tab order (menu → map → sidebar → lists).
- **Enter/Space** activate focused satellite on map/sky plot where click works today.
- Dialogs: default button, Escape closes when safe; don’t trap focus except modal workflows.

## Assistive tech (custom controls)

- `WorldMapControl` / `SkyPlotControl`: set `AutomationProperties.Name` and live region or name updates for **focused satellite** (name, az, el).
- Prefer **ToolTip** on map dots matching the visible label string.

## Maps and imagery

- World map texture may stay light in dark theme; **labels and vectors** must still meet contrast rules.
- Don’t convey pass quality only by footprint fill tint; list/sidebar shows pass details in text.

## Review checklist (PRs touching UI)

1. Identifiable without color?  
2. Contrast checked in **light and dark**?  
3. No critical copy under 12px / low opacity?  
4. Keyboard path exists?  
5. New colors added to shared palette, not inline hex?

See `docs/ACCESSIBILITY.md` for rationale and test notes.

---
> Source: [magicbug/OscarWatch-Tracker](https://github.com/magicbug/OscarWatch-Tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->
