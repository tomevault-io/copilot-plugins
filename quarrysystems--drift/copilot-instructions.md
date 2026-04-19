## drift

> 1. **DRY (Don't Repeat Yourself)** - Extract shared logic into reusable functions/modules


# Code Quality Principles

## Core Principles
1. **DRY (Don't Repeat Yourself)** - Extract shared logic into reusable functions/modules
2. **Single Responsibility Principle** - Each file/function/class should have one clear purpose
3. **File Size Management** - Keep files manageable and logical; split into multiple files when exceeding ~300 lines or mixing concerns

## Code Organization
4. **Imports at Top** - All imports must be at the top of the file, never inline
5. **Logical Grouping** - Group related code with clear section comments
6. **Consistent Naming** - Use descriptive, consistent naming conventions (camelCase for variables/functions, PascalCase for types/classes)

## TypeScript Specific
7. **Explicit Types** - Prefer explicit type annotations over implicit `any`
8. **Interface Over Type** - Use interfaces for object shapes, types for unions/primitives
9. **No Implicit Any** - All parameters and return types should be typed

## Documentation
10. **Preserve Comments** - Do not add or remove comments unless explicitly requested
11. **TSDoc for Public APIs** - Document exported functions/types with TSDoc comments

## Error Handling
12. **Explicit Error Handling** - Handle errors explicitly, avoid silent failures
13. **Meaningful Error Messages** - Include context in error messages

## Testing & Safety
14. **No Breaking Changes** - Avoid changes that break existing functionality without explicit approval
15. **Minimal Edits** - Prefer focused, minimal changes over large refactors

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/QuarrySystems) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
