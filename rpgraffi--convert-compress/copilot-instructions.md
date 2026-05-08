## swift

> - bundle identifier: raffistudio.image-tools


# Swift 6.2 / macOS 14.6+

## Information
- bundle identifier: raffistudio.image-tools

## Architecture & State
- Prefer `struct` over `class` for data models (value semantics prevent shared mutable state)
- Design stateless, pure functions where possible
- Use `@State`, `@Observable`, `@Binding` appropriately; keep state minimal and localized
- Pass dependencies explicitly; avoid singletons except for true app-wide services
- Model state transitions with enums (associated values for data)

## Code Quality
- Single Responsibility: each type/function does one thing well
- Small, focused functions (< 20 lines ideal)
- Immutable by default: use `let` over `var`
- Safe optional handling: use `if let`, `guard let`, or nil coalescing—never force unwrap in production code
- Leverage Swift's type system: use enums, protocols, and generics for compile-time safety
- Explicit > Implicit

## SwiftUI Patterns
- Extract reusable components into separate views
- Keep view bodies readable: extract complex logic to computed properties or ViewModels
- Use `@ViewBuilder` for conditional/compositional layouts
- Prefer declarative over imperative patterns

## General
- Follow Swift API Design Guidelines for naming
- Don't overengineer; prefer simplicity
- Research current APIs and patterns when uncertain


## Folders
- Use `convert-compress/Core/Models` for models
- Use `convert-compress/Core/Constants` for constants
- 

---
> Source: [rpgraffi/convert-compress](https://github.com/rpgraffi/convert-compress) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
