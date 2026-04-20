## parsa-mobile

> This rule documents the custom extension methods provided in [lib/core/extensions](mdc:lib/core/extensions), which add utility functions to core Dart types for improved code clarity and reusability.

# Dart/Flutter Extensions Guide

This rule documents the custom extension methods provided in [lib/core/extensions](mdc:lib/core/extensions), which add utility functions to core Dart types for improved code clarity and reusability.

## Extension Files and Their Purpose

- [color.extensions.dart](mdc:lib/core/extensions/color.extensions.dart):
  - Adds methods to the `Color` class for:
    - Creating a `Color` from a hex string
    - Converting a `Color` to a hex string
    - Lightening or darkening a color
  - **Use when:** You need to parse, display, or adjust colors dynamically in the UI.

- [lists.extensions.dart](mdc:lib/core/extensions/lists.extensions.dart):
  - Adds methods to `List` and `Iterable` for:
    - Selecting a random item
    - Finding the intersection between lists
    - Printing a formatted string from a list of strings
  - **Use when:** You need advanced list operations, random selection, or formatted output for lists.

- [numbers.extensions.dart](mdc:lib/core/extensions/numbers.extensions.dart):
  - Adds methods to `num` for:
    - Formatting numbers as human-readable file sizes (e.g., bytes to kB/MB)
  - **Use when:** Displaying file sizes or data usage to users in a readable format.

- [string.extension.dart](mdc:lib/core/extensions/string.extension.dart):
  - Adds methods to `String` and `String?` for:
    - Capitalizing the first letter
    - Checking if a string is null or empty
    - Safely retrieving a non-empty string or null
  - **Use when:** You need to format, validate, or safely handle strings in UI or logic.

## When to Use Extensions
- Use these extensions to make code more readable and concise by attaching utility methods directly to core types.
- Prefer extensions over utility classes for type-specific operations (e.g., string formatting, color manipulation).
- Use them to avoid code duplication and to centralize common logic.

**See the referenced files for implementation details and example usages.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lazaroborges) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
