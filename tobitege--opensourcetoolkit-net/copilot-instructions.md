## opensourcetoolkit-net

> - ONLY call `dotnet` commands if requested by user.

# Agent Rules

- ONLY call `dotnet` commands if requested by user.

## Avalonia UI Rules

- **RelayCommand CanExecute**: When creating a `RelayCommand` with a `CanExecute` condition (e.g. `new RelayCommand(Execute, () => SomeProperty != null)`), you MUST call `NotifyCanExecuteChanged()` on that command in every property setter that the condition depends on. Failure to do this will leave buttons permanently disabled!
- When the user mentions `glyphs`, use `PathIcon` with the appropriate data attribute.
- Avalonia's compiled bindings require an `x:DataType` on the `DataTemplate` so it knows the item type. Thus add `x:DataType="vm:DataTypeItem"` to the template.
- **UI Composition**: Avoid rigid mutual exclusion in ViewModel setters. Use computed properties (e.g. `IsVisible => EnableZoom || ShowComparison`) to drive UI visibility.
- **Visual Feedback**: Explicitly style active buttons (e.g. `Classes.Add("accent")`) to indicate state; do not rely on default button appearance.
- **Window Sizing**: Use conservative default dimensions (e.g. 600x500) with explicit `MinWidth`/`MinHeight` to support high-DPI scaling.
- **Clipboard Access**: Do not access clipboard directly from ViewModel. Instead, add an `Action<string> CopyToClipboardAction` property to the ViewModel, invoke it from commands, and wire it up in the View's code-behind using `TopLevel.GetTopLevel(this)?.Clipboard?.SetTextAsync(text)`.
- **Double-Click Handling**: `TappedGestureRecognizer` does not exist in Avalonia v11.x. Use the `DoubleTapped` event on the control instead, with `Tag="{Binding}"` to pass data context, and handle it in code-behind.
- **App-Wide Styles**: For common control settings (e.g., `VerticalContentAlignment="Top"` for TextBox), add app-wide styles in `App.axaml` under `<Application.Styles>` rather than repeating them on individual controls. This ensures consistency and reduces duplication.
- **Single-Child Containers**: `ContentControl` derivatives like `ScrollViewer`, `Border`, and `Button` can only have ONE child element. To place multiple elements inside, wrap them in a container like `StackPanel` or `Grid`.
- **ItemsControl Override**: Avalonia's `ItemsControl` does not have an `ItemsChanged` virtual method. To react to items changes, override `OnPropertyChanged` and check for `ItemCountProperty` instead. Use `VisualTreeAttachmentEventArgs` from `Avalonia.VisualTree` namespace for `OnAttachedToVisualTree`.

## Theme Rules

- **Icons Use StaticResource**: `PathIcon.Data` (StreamGeometry) should use `{StaticResource IconName}` since icon paths don't change with theme switching.
- **ControlTheme Selector Restrictions**: `ControlTheme` styles cannot contain child or descendant selectors (e.g. `^[Property=Value] ChildControl`). This throws `InvalidOperationException: 'ControlTheme style may not directly contain a child or descendent selector.'` To fix this:
  1. Change the root element from `<ResourceDictionary>` to `<Styles>`
  2. Wrap `ControlTheme` definitions inside `<Styles.Resources>...</Styles.Resources>`
  3. Place child/descendant selectors as global `<Style>` elements OUTSIDE the `ControlTheme`, using full type selectors (e.g. `controls|MyControl[Property=Value] ChildControl`)
- **Border Has No Foreground**: `Border` does not have a `Foreground` property. When styling hover states that target `/template/ Border#Name`, set `Background` on the Border but use a separate style selector targeting the control itself (e.g. `^:pointerover`) for `Foreground` changes.

### Theming Pitfalls

- **Duplicate Setter Exception**: Do NOT set both `Foreground` and `TextElement.Foreground` on the same element in styles - this causes `System.InvalidOperationException: Duplicate setter encountered for property 'Foreground'`.
- **Always Use DynamicResource for Theme Colors**: `{StaticResource Daisy*Brush}` will NOT update when theme changes at runtime. Always use `{DynamicResource Daisy*Brush}`.
- **TextBlock Foreground Fallback**: If a global `TextBlock` style in `App.axaml` doesn't apply in certain contexts (TabItem headers, DataGrid cells, control templates), add explicit `Foreground="{DynamicResource DaisyBaseContentBrush}"` to those TextBlocks.

## Flowery.NET Usage

This project uses **Flowery.NET** for theming and controls. The theming system uses `DaisyThemeManager` for runtime theme switching.

### Namespace Declaration

```xml
xmlns:controls="clr-namespace:Flowery.Controls;assembly=Flowery.NET"
```

Or use the shorter alias commonly seen:

```xml
xmlns:daisy="clr-namespace:Flowery.Controls;assembly=Flowery.NET"
```

### Dynamic Resource Keys

**Always use Daisy resource keys directly** (not aliases) for proper runtime theme updates:

| Resource Key | Purpose |
|-------------|---------|
| `DaisyBase100Brush` | Primary background |
| `DaisyBase200Brush` | Secondary background (cards, sidebar) |
| `DaisyBase300Brush` | Tertiary background, borders |
| `DaisyBaseContentBrush` | Primary text color |
| `DaisyNeutralBrush` | Neutral fill |
| `DaisyNeutralContentBrush` | Secondary text color |
| `DaisyPrimaryBrush` | Primary accent color |
| `DaisySecondaryBrush` | Secondary accent |
| `DaisyAccentBrush` | Tertiary accent |
| `DaisyInfoBrush` | Info status color |
| `DaisySuccessBrush` | Success status color |
| `DaisyWarningBrush` | Warning status color |
| `DaisyErrorBrush` | Error status color |

Example:

```xml
<Border Background="{DynamicResource DaisyBase200Brush}"
        BorderBrush="{DynamicResource DaisyBase300Brush}">
    <TextBlock Foreground="{DynamicResource DaisyBaseContentBrush}" Text="Hello"/>
</Border>
```

### DaisyButton

Use `DaisyButton` instead of standard `Button` for themed buttons:

```xml
<!-- Color Variants -->
<controls:DaisyButton Content="Primary" Variant="Primary"/>
<controls:DaisyButton Content="Secondary" Variant="Secondary"/>
<controls:DaisyButton Content="Accent" Variant="Accent"/>
<controls:DaisyButton Content="Success" Variant="Success"/>
<controls:DaisyButton Content="Warning" Variant="Warning"/>
<controls:DaisyButton Content="Error" Variant="Error"/>
<controls:DaisyButton Content="Ghost" Variant="Ghost"/>
<controls:DaisyButton Content="Link" Variant="Link"/>

<!-- Button Styles -->
<controls:DaisyButton ButtonStyle="Outline" Variant="Primary" Content="Outline"/>
<controls:DaisyButton ButtonStyle="Dash" Variant="Primary" Content="Dashed"/>
<controls:DaisyButton ButtonStyle="Soft" Variant="Primary" Content="Soft"/>

<!-- Sizes -->
<controls:DaisyButton Size="ExtraSmall" Content="XS"/>
<controls:DaisyButton Size="Small" Content="Small"/>
<controls:DaisyButton Size="Medium" Content="Medium"/>
<controls:DaisyButton Size="Large" Content="Large"/>

<!-- Shapes -->
<controls:DaisyButton Shape="Wide" Content="Wide"/>
<controls:DaisyButton Shape="Square"><PathIcon Data="..." /></controls:DaisyButton>
<controls:DaisyButton Shape="Circle"><PathIcon Data="..." /></controls:DaisyButton>
<controls:DaisyButton Shape="Block" Content="Full Width"/>
```

### DaisyInput (TextBox replacement)

```xml
<controls:DaisyInput Watermark="Enter text..." />
<controls:DaisyInput Variant="Primary" Watermark="Primary style"/>
<controls:DaisyInput Variant="Ghost" Watermark="Ghost style"/>
<controls:DaisyInput Variant="Error" Watermark="Error state"/>
<controls:DaisyInput Size="Small" Watermark="Small input"/>
```

### DaisyTextArea (Multiline TextBox)

```xml
<controls:DaisyTextArea Watermark="Enter description..." Height="100"/>
<controls:DaisyTextArea Variant="Primary" Watermark="Primary style" Height="80"/>
```

### DaisySelect (ComboBox replacement)

```xml
<controls:DaisySelect PlaceholderText="Select an option">
    <ComboBoxItem>Option 1</ComboBoxItem>
    <ComboBoxItem>Option 2</ComboBoxItem>
</controls:DaisySelect>
```

### DaisyCheckBox and DaisyToggle

```xml
<controls:DaisyCheckBox Content="Remember me" Variant="Primary"/>
<controls:DaisyToggle Content="Enable feature" Variant="Primary" IsChecked="True"/>
```

### DaisyCard

```xml
<controls:DaisyCard Width="400">
    <StackPanel Spacing="10">
        <TextBlock Text="Card Title" FontWeight="Bold"/>
        <TextBlock Text="Card content goes here"/>
    </StackPanel>
</controls:DaisyCard>
```

### DaisyDivider

```xml
<controls:DaisyDivider />
```

### Theme Selection Controls

```xml
<!-- Dropdown theme selector -->
<controls:DaisyThemeDropdown Width="180"/>

<!-- Toggle between light/dark -->
<controls:DaisyThemeController Mode="Toggle"/>
<controls:DaisyThemeController Mode="Checkbox"/>
<controls:DaisyThemeController Mode="Swap"/>
<controls:DaisyThemeController Mode="ToggleWithIcons"/>

<!-- Radio button theme selection -->
<controls:DaisyThemeRadio Content="Light" ThemeName="Light" GroupName="Themes"/>
<controls:DaisyThemeRadio Content="Dark" ThemeName="Dark" GroupName="Themes"/>
```

### Color Picker Controls

```xml
<controls:DaisyColorWheel Color="{Binding SelectedColor, Mode=TwoWay}"/>
<controls:DaisyColorEditor Color="{Binding SelectedColor, Mode=TwoWay}" 
                           ShowAlphaChannel="False"/>
```

### Programmatic Theme Switching

```csharp
using Flowery.Controls;

// Apply a theme by name
DaisyThemeManager.ApplyTheme("Synthwave");

// Check current theme
string currentTheme = DaisyThemeManager.CurrentThemeName;

// Listen for theme changes
DaisyThemeManager.ThemeChanged += (sender, themeName) => {
    // Handle theme change
};
```

### App-Wide Styling Guide

This section defines consistent styling patterns for the application.

#### Container Patterns

| Container | Use Case | Component |
|-----------|----------|-----------|
| **Highlighted Section** | Summary cards, results, footers | `<daisy:DaisyCard Variant="Compact">` |
| **Input Form Section** | Form backgrounds | `<Border Background="{DynamicResource DaisyBase200Brush}">` |
| **Plain Container** | DataGrid wrappers | `<Border>` (no background) |

#### Text Color Patterns

| Pattern | Use Case | Brush |
|---------|----------|-------|
| **Standard Text** | Labels, values on Base backgrounds | `DaisyBaseContentBrush` |
| **Success Values** | Positive results, savings | `DaisySuccessBrush` |
| **Warning Values** | Caution, extra costs | `DaisyWarningBrush` |
| **Error Text** | Error messages | `DaisyErrorBrush` |
| **Primary Highlight** | Emphasized values | `DaisyPrimaryBrush` |

> [!WARNING]
> **Never use `DaisyNeutralContentBrush` for text on Base backgrounds!** It's designed for text on dark Neutral backgrounds and causes poor contrast in light themes.

#### Control Sizing

Global defaults in `App.axaml` set all Daisy controls to `Size="Small"` for compact UI:

```xml
<Style Selector="daisyControls|DaisyInput">
    <Setter Property="Size" Value="Small"/>
</Style>
<Style Selector="daisyControls|DaisyButton">
    <Setter Property="Size" Value="Small"/>
</Style>
<Style Selector="daisyControls|DaisySelect">
    <Setter Property="Size" Value="Small"/>
</Style>
```

Standard `NumericUpDown` with `MinWidth="120"` is the gold standard for numeric inputs.

#### Custom Controls

| Control | Purpose | Based On |
|---------|---------|----------|
| `CopyableTextBox` | Multi-line copyable text | DaisyTextArea |
| `CopyableInput` | Single-line copyable value (compact) | DaisyInput |
| `CollapsibleSection` | Expandable content area | DaisyCard + DaisyButton |
| `NumericEntry` | Numeric input with validation | DaisyInput |

#### Hardcoded Colors

**Avoid hardcoded colors!** Use semantic Daisy resources instead:

| ❌ Avoid | ✅ Use Instead |
|----------|---------------|
| `Foreground="Red"` | `Foreground="{DynamicResource DaisyErrorBrush}"` |
| `Foreground="Green"` | `Foreground="{DynamicResource DaisySuccessBrush}"` |
| `Foreground="Orange"` | `Foreground="{DynamicResource DaisyWarningBrush}"` |

**Exception:** `Foreground="White"` is acceptable on colored backgrounds (badges, buttons with Primary/Success/Warning variants).

### Gallery Reference

The `flowery` junction folder's "Gallery" folder contains the official Flowery.NET demo files with examples for ALL controls. Refer to these files for additional patterns:

- `ActionsExamples.axaml` - DaisyButton, Modal, FAB, Swap
- `DataInputExamples.axaml` - DaisyInput, DaisySelect, DaisyCheckBox, DaisyToggle, DaisyRange
- `ThemingExamples.axaml` - Theme controllers and CSS converter
- `CardsExamples.axaml` - DaisyCard variants
- `FeedbackExamples.axaml` - DaisyAlert, DaisyToast, DaisyProgress

---
> Source: [tobitege/OpenSourceToolkit.NET](https://github.com/tobitege/OpenSourceToolkit.NET) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
