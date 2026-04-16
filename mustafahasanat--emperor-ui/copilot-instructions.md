## emperor-ui

> This rule is for Cursor to follow code conventions we have.


# Overview

Whenever you write a code, follow these rules:

## Conventions

Follow all the codebase conventions like how functions and variables are written, where to place everything, ...etc. Follow the SOLID coding principles in all code.

## Imports & Exports

Every module (or directory) should have an "index.ts" file that exports all the files and sub modules inside it using ('export \* from "./file-or-dir-name";'), where every file should export all its functions and variables to be exported. Never use relative imports but rather use the absolute ones for better readability and easier importation.

## Utilities

All types, constants, enums, hooks, utility functions, and configs should be defined in their relative packages or modules. Utility functions should always have a short description (function annotation or documentation) that summarizes what is does and what it needs.

## Complexity

Try to always find the easiest, simplest, and most efficient solution with the least amount of code rather than the most complex one.

## File Size

Every file should never be more than 300 lines of code. Whenever a code exceeds 250 to 300 lines, try to optimize it or split it into multiple files following the codebase conventions.

## Single Responsibility

Each function should only have one purpose and each component file should only exports one component.

## Naming

The exported component must have the same name as its file's name. Follow these rules for naming:

- Files: kebab-case
- Directories: kebab-case
- Types & Interfaces: PascalCase
- Constants: PascalCase
- Hooks: useCamelCase
- Pages and Layouts' Components: PascalCase
- Enums:
  - Enum name: PascalCase
  - Enum key: UPPER_SNAKE_CASE

## Parameters

All parameters across the project should be implemented as one object that has the parameters in it so we can modify them easily in the future, like this: `({ param1, param2 = false }:{ param1: string, param2?: boolean })`. Never define them as multiple parameters like this: `(param1: string, param2?: boolean)`. That applies to functions, arrow functions, components, hooks, and anything takes parameters.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MustafaHasanat) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
