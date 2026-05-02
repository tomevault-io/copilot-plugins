## flowery

> Rules for using Flowery.NET controls

# Flowery.NET

## General Rules

- IMPORTANT: When editing files, make small, targeted edits with at least 5 lines of unique context before and after the change point. Avoid large multi-line replacements; prefer multiple smaller edits.
- Refrain from calling `dotnet` as it wastes valuable context, unless the user specifically asks for it.
- **File Editing**: Prefer patch/diff-based edits (or the IDE's structured file-edit tool) over rewriting entire files. Ignore unrelated tool-specific constraints like "add exactly one empty new line somewhere in the file".
- **Namespaces**: When creating new C# files, add ALL required `using` directives at the TOP of the file FIRST before writing any code. Use fully-qualified namespace imports (e.g. `using Avalonia.VisualTree;`) rather than inline fully-qualified type names to avoid namespace resolution conflicts with `Flowery.*`.

## CODE REQUIREMENTS

For the documentation generator to correctly extract metadata, code must follow
these conventions:

C# CONTROL FILES (Flowery.NET/Controls/Daisy*.cs):

1. Class XML documentation must immediately precede the class definition:

   /// <summary>
   /// A Button control styled after DaisyUI's Button component.
   /// </summary>
   public class DaisyButton : Button

2. StyledProperty definitions must use this exact pattern:

   public static readonly StyledProperty<TYPE> NAMEProperty =
       AvaloniaProperty.Register<CLASS, TYPE>(nameof(NAME), DEFAULT);

3. Property XML documentation must immediately precede the StyledProperty:

   /// <summary>
   /// Gets or sets the button variant (Primary, Secondary, etc.).
   /// </summary>
   public static readonly StyledProperty<DaisyButtonVariant> VariantProperty = ...

4. Enums must be defined at namespace level with public access:

   public enum DaisyButtonVariant
   {
       Default,
       Primary,
       Secondary,
       ...
   }

AXAML EXAMPLE FILES (Flowery.NET.Gallery/Examples/*Examples.axaml):

1. Each control section must start with a SectionHeader:

   <local:SectionHeader SectionId="button" Title="Button" />

2. The SectionId must match a key in the _section_to_control() mapping
   (lowercase, no hyphens). Add new mappings if creating new controls.

3. Sub-examples should be labeled with a TextBlock having FontWeight="SemiBold":

```axaml
   <TextBlock Text="Colors" FontWeight="SemiBold" FontSize="14" Opacity="0.8"/>
   <WrapPanel>
       <controls:DaisyButton Variant="Primary" Content="Primary"/>
       ...
   </WrapPanel>
```

4. Sections are separated by DaisyDivider:

   <controls:DaisyDivider />

5. Control elements use the "controls:" namespace prefix:

   xmlns:controls="clr-namespace:Flowery.Controls;assembly=Flowery.NET"

## ADDING NEW CONTROLS

**For the complete workflow and checklist, also see:** `.cursor/rules/new-control.mdc`

1. Create the C# control file following the patterns above
2. Add examples in the appropriate *Examples.axaml file
3. Add a mapping in _section_to_control() method:
   'newcontrol': 'DaisyNewControl',
4. Run: python Utils/generate_docs.py

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
- **FluentTheme Button Border Is On ContentPresenter (Not Border)**: In Avalonia FluentTheme, `Button` and `ToggleButton` templates use `ContentPresenter#PART_ContentPresenter` and set border-related properties (`BorderBrush`, `BorderThickness`) on that presenter for states like `:pointerover` / `:pressed` / `:checked:pointerover`.
  - If your override targets `/template/ Border`, it will do nothing (and debug colors won’t show), because there often is **no** `Border` in the template.
  - To override hover borders **scoped to a parent control** (e.g. fix button-group dividers only inside `DaisyButtonGroup`), target the template part directly and bind it back to the control’s border brush:

```axaml
<Style Selector="controls|DaisyButtonGroup > :is(Button):pointerover /template/ ContentPresenter#PART_ContentPresenter">
  <Setter Property="BorderBrush" Value="{Binding BorderBrush, RelativeSource={RelativeSource TemplatedParent}}" />
</Style>

<!-- ToggleButton also needs the checked/indeterminate hover selectors to beat Fluent specificity -->
<Style Selector="controls|DaisyButtonGroup > :is(ToggleButton):checked:pointerover /template/ ContentPresenter#PART_ContentPresenter">
  <Setter Property="BorderBrush" Value="{Binding BorderBrush, RelativeSource={RelativeSource TemplatedParent}}" />
</Style>
<Style Selector="controls|DaisyButtonGroup > :is(ToggleButton):indeterminate:pointerover /template/ ContentPresenter#PART_ContentPresenter">
  <Setter Property="BorderBrush" Value="{Binding BorderBrush, RelativeSource={RelativeSource TemplatedParent}}" />
</Style>
```

- **AVLN2000 Nested Selector**: If you hit `AVLN2000: Cannot find parent style for nested selector`, it usually means you used a *nested selector* (e.g. starting with `^` or `:pointerover`) without a parent `<Style>`.
  - Prefer a **full selector** when writing styles directly inside a `Styles` collection (e.g. `ToggleButton:checked PathIcon#LikeIcon`).
  - Or wrap nested selectors under a parent style:

```axaml
<Control.Styles>
  <Style Selector="ToggleButton">
    <Style Selector="^:checked PathIcon#LikeIcon">
      <Setter Property="Data" Value="{StaticResource DaisyIconStarFilled}" />
    </Style>
  </Style>
</Control.Styles>
```

- **ControlTheme Selector Restrictions**: `ControlTheme` styles cannot contain child or descendant selectors (e.g. `^[Property=Value] ChildControl`). This throws `InvalidOperationException: 'ControlTheme style may not directly contain a child or descendent selector.'` To fix this:
  1. Change the root element from `<ResourceDictionary>` to `<Styles>`
  2. Wrap `ControlTheme` definitions inside `<Styles.Resources>...</Styles.Resources>`
  3. Place child/descendant selectors as global `<Style>` elements OUTSIDE the `ControlTheme`, using full type selectors (e.g. `controls|MyControl[Property=Value] ChildControl`)
- **Border Has No Foreground**: `Border` does not have a `Foreground` property. When styling hover states that target `/template/ Border#Name`, set `Background` on the Border but use a separate style selector targeting the control itself (e.g. `^:pointerover`) for `Foreground` changes.

## Avalonia Clipboard Usage

### Image Clipboard (Windows-only)

Avalonia's built-in clipboard API (`DataObject`, `SetDataObjectAsync`) is deprecated and doesn't reliably copy images. For Windows, use **WinForms interop**:

```csharp
using System.Runtime.Versioning;

[SupportedOSPlatform("windows")]
private static void SetBitmapClipboardData(byte[] pngBytes)
{
    if (pngBytes == null || pngBytes.Length == 0) return;
    using var stream = new MemoryStream(pngBytes);
    using var image = System.Drawing.Image.FromStream(stream);
    System.Windows.Forms.Clipboard.SetImage(image);
}
```

### Text Clipboard (Cross-platform)

For text, use Avalonia's built-in clipboard:

```csharp
var clipboard = TopLevel.GetTopLevel(this)?.Clipboard;
if (clipboard != null)
    await clipboard.SetTextAsync(textContent);
```

### Cross-Platform Project Setup

When a project needs WinForms clipboard on Windows but must remain buildable on other platforms:

1. **Conditional TFM** in `.csproj`:

   ```xml
   <TargetFramework Condition="$([MSBuild]::IsOSPlatform('Windows'))">net8.0-windows</TargetFramework>
   <TargetFramework Condition="!$([MSBuild]::IsOSPlatform('Windows'))">net8.0</TargetFramework>
   ```

2. **Conditional WinForms** in `.csproj`:

   ```xml
   <PropertyGroup Condition="$([MSBuild]::IsOSPlatform('Windows'))">
     <UseWindowsForms>true</UseWindowsForms>
     <DefineConstants>$(DefineConstants);WINDOWS</DefineConstants>
   </PropertyGroup>
   ```

3. **Conditional compilation** in code:

   ```csharp
   #if WINDOWS
   if (OperatingSystem.IsWindows())
   {
       SetBitmapClipboardData(pngBytes);
   }
   else
   #endif
   {
       // Fallback: save to temp file, copy path to clipboard
       var tempPath = Path.Combine(Path.GetTempPath(), "screenshot.png");
       await File.WriteAllBytesAsync(tempPath, pngBytes);
       await clipboard.SetTextAsync(tempPath);
   }
   ```

**Key points:**

- `UseWindowsForms` requires `net8.0-windows` TFM (SDK enforced)
- Use `#if WINDOWS` preprocessor directives to guard WinForms code
- Mark Windows-specific methods with `[SupportedOSPlatform("windows")]`
- Always provide a fallback for non-Windows platforms

## File/Folder Dialogs (StorageProvider API)

The old `SaveFileDialog`, `OpenFileDialog`, and `OpenFolderDialog` classes are **deprecated** in Avalonia 11. Use the modern `StorageProvider` API instead.

### Required Import

```csharp
using Avalonia.Platform.Storage;
```

### Save File Dialog

```csharp
var topLevel = TopLevel.GetTopLevel(this);
var file = await topLevel.StorageProvider.SaveFilePickerAsync(new FilePickerSaveOptions
{
    Title = "Save File",
    SuggestedFileName = "myfile.png",
    DefaultExtension = "png",
    FileTypeChoices = new[]
    {
        new FilePickerFileType("PNG Images") { Patterns = new[] { "*.png" } },
        new FilePickerFileType("All Files") { Patterns = new[] { "*.*" } }
    }
});

if (file != null)
{
    var filePath = file.Path.LocalPath;
    // Use filePath...
}
```

### Open File Dialog

```csharp
var files = await topLevel.StorageProvider.OpenFilePickerAsync(new FilePickerOpenOptions
{
    Title = "Select File",
    AllowMultiple = false,
    FileTypeFilter = new[]
    {
        new FilePickerFileType("Images") { Patterns = new[] { "*.png", "*.jpg" } }
    }
});

if (files.Count > 0)
{
    var filePath = files[0].Path.LocalPath;
    // Use filePath...
}
```

### Folder Picker Dialog

```csharp
var folders = await topLevel.StorageProvider.OpenFolderPickerAsync(new FolderPickerOpenOptions
{
    Title = "Select Folder",
    AllowMultiple = false
});

if (folders.Count > 0)
{
    var folderPath = folders[0].Path.LocalPath;
    // Use folderPath...
}
```

**Key points:**

- Access via `TopLevel.GetTopLevel(control).StorageProvider`
- Returns `IStorageFile` / `IStorageFolder` objects; use `.Path.LocalPath` for string path
- `SaveFilePickerAsync` returns `null` if cancelled
- `OpenFilePickerAsync` / `OpenFolderPickerAsync` return empty list if cancelled

---
> Source: [tobitege/OpenSourceToolkit.NET](https://github.com/tobitege/OpenSourceToolkit.NET) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
