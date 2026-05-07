## swift

> Swift and SwiftUI coding standards and style guidelines


# Swift Style Guide

Swift and SwiftUI coding standards and style guidelines for our iOS codebase. These guidelines ensure consistency, maintainability, and high code quality across Swift projects.

## Core Principles

- **KISS (Keep It Simple, Stupid)** - Always choose the simplest, most maintainable solution
- **SwiftUI First** - Always prefer SwiftUI over UIKit when possible
- **Less is More** - Always avoid unnecessary complexity, the best code is no code
- **Self-Documenting** - Always make code obvious and clear without comments

## Linter Errors and False Positives

### Ignoring False Positive Errors

**Always ignore false positive linter errors from Cursor/VS Code** when working with Swift code. The Swift language server in VS Code/Cursor cannot properly resolve Swift symbols and dependencies that are correctly configured in Xcode.

**Common false positive errors to ignore:**

- `Cannot find 'X' in scope` - When X is clearly defined in the codebase
- `Reference to member 'X' cannot be resolved without a contextual type` - When X is a valid SwiftUI/UIKit type
- `No such module 'X'` - When the module is correctly imported in Xcode
- `Value of type 'X' has no member 'Y'` - When Y is a valid member in Xcode
- Any reference errors for custom views, models, or utilities that compile successfully in Xcode

**Workflow:**

- Use Cursor for AI assistance and code generation
- Use Xcode for actual development and compilation
- Only report real compilation errors from Xcode, not linter errors from Cursor/VS Code
- Trust Xcode's build system over Cursor's language server for Swift

## File Organization

### Directory Structure

- Always organize code in a predictable and scalable way
- Always keep related code close together
- Always use clear, descriptive directory names
- Always follow consistent patterns across the project
- Always use singular for categories/domains (e.g. `Feature/`, `Model/`, `View/`)
- Always use plural for collections/lists (e.g. `Features/`, `Models/`, `Views/`)

✅ Good:

```swift
Tapling/
  Features/
    Collectible/
      Environment/
        Models/
        CollectibleRegistry.swift
      Views/
  Routes/
    Settings/
      SettingsView.swift
  Views/
    ActionRowView.swift
```

❌ Bad:

```swift
Tapling/
  feature/           // Wrong: Should be Features (plural)
    collectible/     // Wrong: Should be Collectible (PascalCase)
      model/         // Wrong: Should be Models (plural)
```

### File Naming

- Always use `PascalCase` for Swift files (e.g. `SettingsView.swift`, `TaplingSettings.swift`)
- Always match the file name to the primary type/struct/class it contains
- Always use descriptive names that indicate purpose

✅ Good:

```swift
SettingsView.swift        // Contains SettingsView struct
TaplingSettings.swift     // Contains TaplingSettings class
ActionRowView.swift       // Contains ActionRowView struct
```

❌ Bad:

```swift
settings.swift           // Wrong: Should be PascalCase
Settings.swift           // Wrong: Too generic
View.swift               // Wrong: Not descriptive
```

## SwiftUI View Structure

### View Organization

- Always follow the 3-layer structure: Variables → UI → Actions
- Always use `// MARK: - UI` to separate UI components from actions
- Always use `// MARK: - Actions` to separate actions from UI
- Never add more than these 2 MARKs - if you need more organization, split the view

✅ Good:

```swift
struct SettingsView: View {
    // Variables (no MARK needed)
    @State private var isEnabled = false
    @QuerySingleton private var settings: Settings

    // MARK: - UI

    var body: some View {
        Form {
            headerSection
        }
    }

    private var headerSection: some View {
        Section {
            Toggle("Enabled", isOn: isEnabledBinding)
        }
    }

    // MARK: - Actions

    private func saveSettings() {
        try? modelContext.save()
    }
}
```

❌ Bad:

```swift
struct SettingsView: View {
    // MARK: - Properties  // Wrong: No MARK for variables
    @State private var isEnabled = false

    // MARK: - UI
    var body: some View { }

    // MARK: - Search UI     // Wrong: Too many MARKs
    // MARK: - Filter UI     // Wrong: Split into separate views instead
    // MARK: - Actions
}
```

### View Components

- Always use computed properties for view sections
- Always use descriptive names ending with `Section`, `View`, or `Button` (e.g. `headerSection`, `settingsForm`, `saveButton`)
- Always keep view components private unless they need to be reused elsewhere

✅ Good:

```swift
private var headerSection: some View {
    VStack {
        Text("Title")
    }
}

private var settingsForm: some View {
    Form {
        // ...
    }
}
```

❌ Bad:

```swift
var header: some View { }        // Wrong: Not descriptive
public var section: some View { } // Wrong: Should be private unless reused
```

## Naming Conventions

### Types and Structures

- Always use `PascalCase` for types, structs, classes, enums, protocols
- Always use descriptive names that clearly indicate purpose
- Always prefix protocols with descriptive names (e.g. `SingletonModel`, not `Model`)

✅ Good:

```swift
struct SettingsView: View { }
class DataContainer { }
protocol SingletonModel { }
enum Rarity { }
```

❌ Bad:

```swift
struct view: View { }           // Wrong: Should be PascalCase
class container { }             // Wrong: Should be PascalCase
protocol Model { }             // Wrong: Too generic
```

### Variables and Functions

- Always use `camelCase` for variables and functions
- Always use descriptive names that indicate purpose
- Always use verb phrases for functions that perform actions
- Always use noun phrases for computed properties

✅ Good:

```swift
private var isEnabled = false
private func saveSettings() { }
private func formatDate(_ date: Date) -> String { }
private var headerSection: some View { }
```

❌ Bad:

```swift
var enabled = false             // Wrong: Should indicate boolean
func save() { }                // Wrong: Too generic
func date(_ d: Date) -> String { } // Wrong: Not a verb phrase
```

### Property Wrappers

- Always use appropriate property wrappers (`@State`, `@Query`, `@QuerySingleton`, `@Environment`, etc.)
- Always make property wrapper variables `private` unless they need external access
- Always use descriptive names that indicate the property's purpose

✅ Good:

```swift
@State private var isEnabled = false
@QuerySingleton private var settings: TaplingSettings
@Environment(\.modelContext) private var modelContext
```

❌ Bad:

```swift
@State var enabled = false     // Wrong: Should be private
@Query var items: [Item]       // Wrong: Should be private
```

## Code Style

### Imports

- Always import only what you need
- Always group imports logically (Foundation, SwiftUI, SwiftData, then third-party)
- Always use explicit imports when possible

✅ Good:

```swift
import Foundation
import SwiftData
import SwiftUI
import KeyboardKit
```

❌ Bad:

```swift
import *                    // Wrong: Not valid Swift
import Foundation.SwiftUI  // Wrong: Incorrect import syntax
```

### Spacing and Formatting

- Always use consistent indentation (spaces, not tabs)
- Always add blank lines between logical sections
- Always use trailing commas in multi-line arrays/dictionaries
- Always format code consistently with Xcode's formatter

✅ Good:

```swift
let items = [
    "item1",
    "item2",
    "item3",
]
```

❌ Bad:

```swift
let items = ["item1","item2","item3"]  // Wrong: No spacing
```

### Error Handling

- Always handle errors explicitly
- Always use `try?` when errors can be safely ignored
- Always use `do-catch` when error handling is important
- Always provide meaningful error messages

✅ Good:

```swift
do {
    try modelContext.save()
} catch {
    print("Failed to save: \(error)")
}

// Or when error can be ignored
try? modelContext.save()
```

❌ Bad:

```swift
try modelContext.save()  // Wrong: Unhandled error
```

## SwiftData Patterns

### Model Definitions

- Always use `@Model` macro for SwiftData models
- Always provide default values where appropriate
- Always use `SingletonModel` protocol for singleton models

✅ Good:

```swift
@Model
final class TaplingSettings: SingletonModel {
    var scale: Double = 1.0
    var position: HandPosition = .center

    static var `default`: TaplingSettings {
        TaplingSettings()
    }
}
```

❌ Bad:

```swift
class TaplingSettings {  // Wrong: Missing @Model
    var scale: Double     // Wrong: No default value
}
```

### Querying Data

- Always use `@Query` for collections
- Always use `@QuerySingleton` for singleton models
- Always make query properties `private`

✅ Good:

```swift
@Query private var collectibles: [OwnedCollectible]
@QuerySingleton private var settings: TaplingSettings
```

❌ Bad:

```swift
@Query var items: [Item]  // Wrong: Should be private
```

## Documentation

### Comments

- Always use comments sparingly - code should be self-documenting
- Always use `///` for documentation comments
- Always explain "why" not "what" in comments

✅ Good:

```swift
/// Ensures singleton models exist in the given context.
/// Creates default instances if they don't exist.
static func ensureSingletons(in context: ModelContext) {
    // ...
}
```

❌ Bad:

```swift
// This function ensures singletons
// It creates default instances
static func ensureSingletons(in context: ModelContext) {
    // ...
}
```

## Testing

### Test Structure

- Always organize tests to match source structure
- Always use descriptive test names that explain what is being tested
- Always use `XCTest` for unit tests

✅ Good:

```swift
func testSettingsDefaultValues() {
    let settings = TaplingSettings.default
    XCTAssertEqual(settings.scale, 1.0)
}
```

❌ Bad:

```swift
func test1() {  // Wrong: Not descriptive
    // ...
}
```

---
> Source: [builder-group/focuscat](https://github.com/builder-group/focuscat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
