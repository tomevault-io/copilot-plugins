## unreal-umg-design-system

> Guidance for AI coding assistants (Copilot, Cursor, Codex, Claude Code) working in a project that consumes the **Unreal UMG Design System** plugin, or in this repo itself.

# AGENTS.md

Guidance for AI coding assistants (Copilot, Cursor, Codex, Claude Code) working in a project that consumes the **Unreal UMG Design System** plugin, or in this repo itself.

## What this project is

A source-only Unreal Engine plugin (module `UmgDesignSystem`, class prefix `Ds`) providing design tokens (`UDsTheme`), themed UMG widgets, a 120-icon SVG set, and runtime theming. It ships **no content assets**; everything is C++.

## Golden rules (do not violate)

1. **Use `Ds` widgets, not stock widgets, for anything user-facing.** `UDsButton` instead of `UButton` + hand styling, `UDsText` instead of `UTextBlock` with a hardcoded font, `UDsTextBox` instead of `UEditableTextBox`, `UDsTextArea` instead of `UMultiLineEditableTextBox`, `UDsDropdown` instead of `UComboBoxString`, `UDsSearchBox` for search fields, `UDsTabs` for tab strips, `UDsToggle`/`UDsCheckBox`/`UDsRadio` for booleans, `UDsRangeSlider` for min/max ranges, `UDsScrollBox` instead of `UScrollBox`, `UDsDragChip`/`UDsDropZone` for drag & drop, and so on. The point of the system is that no screen carries literal colors, sizes, or fonts.
2. **Never hardcode a color, radius, spacing, or font size.** If you need a raw value (custom Slate work, a brush for a stock widget), resolve the theme and read tokens: `const UDsTheme* T = UDsTheme::Resolve(Widget);` then `T->Primary.Base`, `T->RadiusMD`, `T->Space4`, `T->MakeFontInfo(EDsTextRole::Body1)`. In Blueprint use `UDsStyleLibrary`.
3. **Do not cache resolved styles across theme changes.** If you build styles manually, rebuild them in a handler subscribed to `FDsThemeEvents::OnThemeChanged` (subscribe while your Slate widget is alive, unsubscribe in `ReleaseSlateResources`). Ds widgets already do this.
4. **Restyling stock `UEditableTextBox` at runtime: never pass a temporary to `SetWidgetStyle`.** UE 5.8's implementation points the live Slate widget at the *argument* (`&InStyle`), not at the copied member — a temporary produces a dangling pointer and a prepass crash one frame later. Keep the `FEditableTextBoxStyle` in a member that outlives the widget. (`UDsTextBox` handles this internally; this rule is for stock-widget styling only. Slider/ProgressBar/CheckBox pass their member and are safe.)
5. **Icons by name, white-fill only.** `UDsIcon` / `UDsButton::Icon` take names like `"chevron_left"`, `"sword"`, `"settings"` (file names under `Resources/Icons/`, no extension). New icons must be white-fill SVGs so the tint cascade works; black-fill icons render black regardless of tint.
6. **Pills are height/2 at draw time.** Do not invent pill-radius tokens; `FDsStyles::MakeRoundedBrush(Fill, Height * 0.5f, ...)` is the pattern.
7. **Variants over custom colors.** A destructive action is `EDsButtonVariant::Danger`, not a red tint on Primary. Status text is a `DsText` tone (`Success`/`Warning`/`Danger`/`Info`), not `SetColorAndOpacity` with a literal.

## API map

| Need | Use |
| --- | --- |
| The active theme (never null) | `UDsTheme::Resolve(WorldContext)` / BP `UDsStyleLibrary::GetResolvedTheme` |
| Swap theme at runtime | `UDsThemeSubsystem::SetActiveTheme` / `SetActiveThemePreset(EDsThemePreset::Dark\|Light)` |
| React to theme swaps (C++) | `FDsThemeEvents::OnThemeChanged` (static multicast) |
| React to theme swaps (BP) | `UDsThemeSubsystem::OnThemeChanged` (assignable) |
| Style a stock widget | `UDsStyleLibrary::MakeButtonStyle / MakeTextStyle / MakeTextBoxStyle / MakeSliderStyle / MakeProgressBarStyle / MakeCardBrush` |
| An icon brush for custom Slate | `UDsIconLibrary::MakeIconBrush(Name, Size, Tint)` |
| All icon names | `UDsIconLibrary::GetIconNames()` |
| An IconName property with a dropdown | `UPROPERTY(... meta = (GetOptions = "/Script/UmgDesignSystem.DsIconLibrary.GetIconNameOptions"))` |
| Rounded/circle brushes from tokens | `FDsStyles::MakeRoundedBrush / MakeCircleBrush` |
| Tabbed panels | `UDsTabs::OnTabChanged` -> `UWidgetSwitcher::SetActiveWidgetIndex` |
| Accept a Ds drag in custom Slate | `DragDropEvent.GetOperationAs<FDsDragDropOp>()` (carries `ItemId` + `SourceChip`) |
| The showcase | console `Ds.Gallery`; args compose: `light`, a palette name (`tokyo`), `end`, `shot`/`shotquit` |

## Repository layout

```
UmgDesignSystem.uplugin          plugin descriptor (EnabledByDefault)
Resources/Icons/*.svg            the 120-icon set (staged NonUFS at package time)
Source/UmgDesignSystem/
  Public/
    DsTypes.h                    every enum (variants, sizes, roles, tones, ...)
    DsTheme.h                    UDsTheme tokens + presets + Resolve() + FDsThemeEvents
    DsThemeSubsystem.h           per-game-instance active theme + change broadcast
    DsSettings.h                 project settings (default theme, extra icon dirs)
    DsStyles.h                   token -> Slate style builders (the stylesheet layer)
    DsIconLibrary.h              icon name registry + vector brush factory
    DsStyleLibrary.h             Blueprint wrappers for the builders
    Slate/SDsButton.h            SButton + press-scale animation
    Slate/SDsToggle.h            custom switch/checkbox leaf widget
    Widgets/Ds*.h                the UMG widgets (palette category "Design System")
  Private/                       implementations + UmgDesignSystemModule.cpp (Ds.Gallery)
docs/                            THEMING / ICONS / WIDGETS / ARCHITECTURE
```

## Conventions when editing the system

- Class prefix `Ds`; one widget per header under `Widgets/`; custom Slate under `Slate/`.
- Every widget follows the same lifecycle contract: subscribe to `FDsThemeEvents::OnThemeChanged` in `RebuildWidget`, apply styles in a private `ApplyDsStyle()` called from `SynchronizeProperties` and the theme handler, unsubscribe + reset Slate pointers in `ReleaseSlateResources`, and override `GetPaletteCategory` (`#if WITH_EDITOR`) to return "Design System".
- Slate style structs handed to Slate by pointer live as stable members of the UObject (`ButtonStyle`, `IconBrush`, `StyleStorage`), never as locals.
- In raw `OnPaint` code, `FSlateDrawElement::MakeBox` does NOT apply the brush's own `TintColor` - multiply `Brush->GetTint(InWidgetStyle)` into the element tint yourself (see `SDsToggle`'s `DrawBrush` helper), or every fill renders white while outlines keep their colors.
- Palette values are authored as hex in `UDsTheme::ApplyPreset` via `Hex(0xRRGGBB)`; the dark preset is the canonical palette (it doubles as the CDO defaults), so change it deliberately and keep the light preset in step.
- Local variables named `Slot` shadow `UUserWidget::Slot` and fail the build (`C4458` + warnings-as-errors); name them `AddedSlot` or similar.

## Validate changes

Compile against a 5.8 project, then run the smoke test the screenshots come from:

```
UnrealEditor-Cmd <project>.uproject -game -windowed -resx=1600 -resy=1000 \
  -nosplash -unattended -nopause -ExecCmds="Ds.Gallery shotquit"
```

Exit code 0 plus `Saved/Screenshots/.../DsGallery.png` showing the gallery = the whole stack works (module load, theme resolve, every widget's construction, icon enumeration + rasterization, paint). Repeat with `Ds.Gallery shotquit light` to exercise the runtime re-theme path — that variant is what caught the `SetWidgetStyle` engine bug.

---
> Source: [sinanata/unreal-umg-design-system](https://github.com/sinanata/unreal-umg-design-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
