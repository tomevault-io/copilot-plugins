## smartcardexplorer

> **NEVER use `git reset --hard`, `git checkout` to discard changes, or any other destructive git commands.**

# Claude Code Instructions for SmartCardExplorer

## Critical Rules

### NEVER ROLLBACK A GIT REPOSITORY
**NEVER use `git reset --hard`, `git checkout` to discard changes, or any other destructive git commands.**

The job of the repository is to protect against mistakes. If something needs to be undone:
- Use `git revert` to create a new commit that undoes changes
- Or manually edit files to fix issues
- NEVER use destructive commands that lose commit history or uncommitted work

### ALWAYS ASK BEFORE COMMITTING
**NEVER commit changes without explicit user approval.**

Before ANY git commit:
1. Verify the build succeeds
2. **ASK THE USER** "Ready to commit?" or similar - wait for their confirmation
3. Only commit after receiving explicit approval

This applies even when the user says "commit everything" - still confirm before executing.
Do NOT assume permission based on context or workflow. Always ask.

### Submodule Policy
**Submodules must NOT be permanently modified to accommodate requirements of the main project.**

Changes to submodules are only permitted for:
- Testing
- Debugging
- Instrumentation

Any functionality needed from a submodule must be implemented in the main project through:
- Event hooks (e.g., PreparePropertyItem)
- Custom editors and converters
- Other extension points

All submodule changes must be reverted before committing.

## Project Notes

### PropertyGrid TypeConverter Support
The PropertyGrid functionality for TypeConverter-generated properties is implemented entirely in the main WPF project, without modifying the WpfExtendedToolkit submodule:

- **MainWindow.xaml.cs OnPreparePropertyItem**: Detects TypeConverter-generated properties, sets PropertyItem.Value directly, and assigns appropriate editors
- **TypedPropertyDescriptor**: Overrides Converter property to return TypeConverter from attributes
- **ReadOnlyTextEditor + PropertyDescriptorValueConverter**: Handles value display using PropertyDescriptor.GetValue()
- **XAML EditorDefinitions**: Custom templates for X509Extension and other types

The WpfExtendedToolkit submodule remains at its original state (commit e9fb4e2).

### Custom PropertyGrid Editors (ExpandableTextBox, etc.)
To add properties that need custom editors (like ExpandableTextBox for large text), only add an EditorTemplateDefinition in **MainWindow.xaml**:

```xml
<toolkit:EditorTemplateDefinition TargetProperties="PropertyName1,PropertyName2">
    <toolkit:EditorTemplateDefinition.EditingTemplate>
        <DataTemplate>
            <Controls:ExpandableTextBox Text="{Binding Value, Mode=OneWay}"/>
        </DataTemplate>
    </toolkit:EditorTemplateDefinition.EditingTemplate>
</toolkit:EditorTemplateDefinition>
```

The code automatically detects EditorDefinitions at runtime and skips applying ReadOnlyTextEditor for those properties. Use C# property names (not DisplayName).

### Expandable Types in PropertyGrid

To make a type expandable in the PropertyGrid with proper indentation and summary display, create TypeConverters in `SmartCardExplorer.Windows.WPF\Converters\`. The PropertyGrid automatically detects expandable types via `TypeConverter.GetPropertiesSupported()`.

#### Single Expandable Items (e.g., PLMN, Oid)

Create a TypeConverter extending `ExpandableObjectConverter`:

```csharp
public class MyTypeConverter : ExpandableObjectConverter
{
    public override bool CanConvertTo(ITypeDescriptorContext context, Type destinationType)
        => destinationType == typeof(string) || base.CanConvertTo(context, destinationType);

    public override object ConvertTo(ITypeDescriptorContext context, CultureInfo culture, object value, Type destinationType)
    {
        if (destinationType == typeof(string) && value is MyType item)
            return item.ToString();  // Summary when collapsed
        return base.ConvertTo(context, culture, value, destinationType);
    }

    public override bool GetPropertiesSupported(ITypeDescriptorContext context) => true;

    public override PropertyDescriptorCollection GetProperties(ITypeDescriptorContext context, object value, Attribute[] attributes)
    {
        if (value is not MyType item)
            return base.GetProperties(context, value, attributes);

        // Use FuncPropertyDescriptor for read-only child properties
        return new PropertyDescriptorCollection(new PropertyDescriptor[]
        {
            new FuncPropertyDescriptor<MyType>("Property1", () => item.Property1),
            new FuncPropertyDescriptor<MyType>("Property2", () => item.Property2)
        });
    }
}
```

#### Collections with Expandable Items (e.g., PLMNCollection, OidCollection)

Create a TypeConverter extending `TypeConverter` (not ExpandableObjectConverter):

```csharp
public class MyCollectionConverter : TypeConverter
{
    private static readonly Attribute[] ItemAttributes = { new TypeConverterAttribute(typeof(MyTypeConverter)) };

    public override bool GetPropertiesSupported(ITypeDescriptorContext context) => true;

    public override PropertyDescriptorCollection GetProperties(ITypeDescriptorContext context, object value, Attribute[] attributes)
    {
        if (value is not MyCollection items)
            return new PropertyDescriptorCollection(Array.Empty<PropertyDescriptor>());

        // Use TypedPropertyDescriptor for each item, with the item's TypeConverter
        var descriptors = items
            .Select((item, i) => new TypedPropertyDescriptor<MyCollection>(
                GetItemName(item, i), item, typeof(MyType), ItemAttributes))
            .ToArray();

        return new PropertyDescriptorCollection(descriptors);
    }

    public override bool CanConvertTo(ITypeDescriptorContext context, Type destinationType)
        => destinationType == typeof(string) || base.CanConvertTo(context, destinationType);

    public override object ConvertTo(ITypeDescriptorContext context, CultureInfo culture, object value, Type destinationType)
    {
        if (destinationType == typeof(string) && value is MyCollection items)
            return items.Count == 0 ? "(none)" : string.Join(", ", items);  // Summary
        return base.ConvertTo(context, culture, value, destinationType);
    }

    private static string GetItemName(MyType item, int index) => item.Name ?? $"Item {index + 1}";
}
```

#### Registration in App.xaml.cs

Register TypeConverters in `OnStartup`:

```csharp
TypeDescriptor.AddAttributes(typeof(MyType), new TypeConverterAttribute(typeof(MyTypeConverter)));
TypeDescriptor.AddAttributes(typeof(MyCollection), new TypeConverterAttribute(typeof(MyCollectionConverter)));
```

**Key points:**
- `GetPropertiesSupported()` returning `true` enables expansion (detected automatically in `OnPreparePropertyItem`)
- Single items: extend `ExpandableObjectConverter`, use `FuncPropertyDescriptor<T>` for child properties
- Collections: extend `TypeConverter`, use `TypedPropertyDescriptor<T>` with item's TypeConverter in attributes
- `ConvertTo()` provides the summary string shown when collapsed
- Examples: `OidTypeConverter`, `PLMNTypeConverter` (single items); `OidCollectionConverter`, `PLMNCollectionConverter` (collections)

### Web Content Fetching
When needing to fetch web content (documentation, specifications, etc.) and the WebFetch tool is blocked or returns poor results, use a local script instead:

```powershell
# PowerShell example
Invoke-WebRequest -Uri "https://example.com/spec.pdf" -OutFile "spec.pdf"
# Or for HTML content:
(Invoke-WebRequest -Uri "https://example.com/page").Content | Out-File "page.html"
```

This avoids LLM-specific blocking and provides first-class access to web resources.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/graealex)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/graealex)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
