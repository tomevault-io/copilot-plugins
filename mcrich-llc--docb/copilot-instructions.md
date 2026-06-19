## docb

> Welcome, agent! When making modifications to this repository on behalf of a human user, please strictly adhere to the following directives:

# Directives for AI Coding Assistants

Welcome, agent! When making modifications to this repository on behalf of a human user, please strictly adhere to the following directives:

## 1. Execute With Verification
After performing a refactor or writing new code, **always** use your Xcode or build tools to dry-run a build and ensure no compiler errors were introduced. Do not ask for permission to verify your own code; proactively fix your errors.

## 2. Adhere to Code Structure
Match the existing architectural patterns (SwiftData models, SwiftUI conventions) natively found in the `DocBCore` package. If you are modifying models, respect `DTO` bridging patterns.

## 3. Intelligent Tool Usage
If doing complex file replacements, prefer your native AST/diff editing tools over brittle bash tools like `sed`. Read files thoroughly to gain context before modifying.

## 4. Documentation Standards

This section details the conventions and best practices for writing code documentation within the `DocB` project. Following these instructions ensures that Xcode's Quick Help and DocC generation provide accurate, cleanly formatted documentation for all contributors.

### 1. General Principles
- Write documentation as complete sentences with proper capitalization and punctuation.
- Summarize the purpose of the symbol in the very first sentence.

### 2. Formatting & Syntax
- **Parameters**: Use the standard Xcode `/// - Parameters:` block, followed by indented `///   - paramName: Description` lines.
- **Return Values**: Use `/// - Returns: Description` to clarify return types if they are non-obvious.
- **Callouts**: Use standard markup callouts like `/// - Important:`, `/// - Note:`, or `/// - Warning:` where additional context is needed.

### 3. Placement Rules
The exact placement of your `///` comments depends on the specific modifiers and decorators preceding your types.

#### A. Global Attributes on Types
When a class or struct uses global attributes like `@Model` or `@MainActor`, the documentation comment should be placed **before** all attributes:
```swift
/// Encapsulates the application's configuration state.
@Model
public final class AppConfiguration {
    // ...
}
```

#### B. Property Wrappers (State, Query, Binding)
For typical property wrappers such as `@State`, `@Binding`, `@Query`, and `@AppStorage`, the documentation comment goes **before** the property wrapper:
```swift
/// The collections saved in SwiftData.
@Query(sort: \.lastUpdatedDate) private var collections: [BookmarkCollection] = []

/// Controls presentation of the creation sheet.
@State private var isShowingSheet = false
```

#### C. Environment Properties
**Do not** document `@Environment` property wrappers. Across the entire project, calls injected from `@Environment` should be left uncommented to reduce clutter, as their usage is typically standardized and obvious by SwiftUI (e.g. `dismiss`, `modelContext`). 
```swift
// Correct
@Environment(\.dismiss) private var dismiss

// Incorrect ❌
/// The dismiss action provided by the environment.
@Environment(\.dismiss) private var dismiss
```

### 4. Visibility Protocol
- Any `struct`, `class`, model, or protocol exposed across the `DocBCore` package boundaries must be marked `public`.
- Ensure that the primary initializer (`public init(...)`) mapping the object's creation is thoroughly documented as well, alongside explicit parameter documentation.

---
> Source: [Mcrich-LLC/DocB](https://github.com/Mcrich-LLC/DocB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
