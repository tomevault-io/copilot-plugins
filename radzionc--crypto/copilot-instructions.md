## reuse-lib

> REUSE existing code WHEN implementing new features FROM @lib packages TO maintain consistency and reduce duplication

# Reuse Lib Code Rules

## Context
- When implementing new features or components
- When adding new types or interfaces
- When creating utility functions

## Requirements
- Always check @lib/ui/props for reusable prop types
- Always check @lib/utils for existing utility functions
- Always check @lib/ui for existing UI components
- Never duplicate types or functions that already exist in lib packages
- Use existing prop types like ChildrenProp, KindProp, etc.
- Use existing utility functions from @lib/utils
- Use existing UI components from @lib/ui

## Examples

<example>
// Instead of defining your own children prop
interface MyComponentProps {
  children: ReactNode
}

// Use the existing ChildrenProp
import { ChildrenProp } from '@lib/ui/props'
interface MyComponentProps extends ChildrenProp {}
</example>

<example>
// Instead of defining your own kind prop
interface MyComponentProps {
  kind: 'primary' | 'secondary'
}

// Use the existing KindProp
import { KindProp } from '@lib/ui/props'
interface MyComponentProps extends KindProp<'primary' | 'secondary'> {}
</example>

<example type="invalid">
// Don't duplicate existing types
interface ButtonProps {
  onClick: () => void
  children: ReactNode
  className?: string
}

// Instead use existing props
import { OnClickProp, ChildrenProp, ClassNameProp } from '@lib/ui/props'
interface ButtonProps extends OnClickProp, ChildrenProp, ClassNameProp {}
</example>

---
> Source: [radzionc/crypto](https://github.com/radzionc/crypto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
