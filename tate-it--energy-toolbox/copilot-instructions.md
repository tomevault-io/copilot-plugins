## energy-toolbox

> Refer to the complete guide: [adding-new-steps-guide.md](mdc:documentation/adding-new-steps-guide.md)

# XML Generator Constants

Refer to the complete guide: [adding-new-steps-guide.md](mdc:documentation/adding-new-steps-guide.md)

## Constants Pattern (MANDATORY)

ALL constants must follow this exact pattern:

```typescript
// Labels for user-facing text (always in Italian)
export const FEATURE_LABELS: Record<string, string> = {
  '01': 'Prima Opzione',
  '02': 'Seconda Opzione', 
  '03': 'Terza Opzione',
  '99': 'Altro',
} as const

// Enum values for type safety
export const FEATURE_VALUES = {
  FIRST: '01',
  SECOND: '02',
  THIRD: '03',
  OTHER: '99',
} as const
```

## Requirements

### Italian Labels (MANDATORY)
- ALL user-facing labels must be in Italian
- Use proper Italian terminology
- Follow existing translation patterns

### Type Safety (REQUIRED)
- Always use `as const` for type safety
- Create both labels (for UI) and enum values (for logic)
- Use `Record<string, string>` for label mappings

### Naming Conventions
- Use `_LABELS` suffix for user-facing text
- Use `_VALUES` for enum constants
- Use SCREAMING_SNAKE_CASE for constant names

### SII Specification Alignment
- Match codes to SII specification requirements
- Use numeric string codes as specified (e.g., '01', '02')
- Include comments referencing SII documentation

## Usage in Components

Components must import and use constants like this:

```typescript
import { FEATURE_LABELS } from '@/lib/xml-generator/constants'

// In component render:
{Object.entries(FEATURE_LABELS).map(([value, label]) => (
  <SelectItem key={value} value={value}>
    {label}
  </SelectItem>
))}
```

Never hardcode option labels directly in components.
# XML Generator Constants

Refer to the complete guide: [adding-new-steps-guide.md](mdc:documentation/adding-new-steps-guide.md)

## Constants Pattern (MANDATORY)

ALL constants must follow this exact pattern:

```typescript
// Labels for user-facing text (always in Italian)
export const FEATURE_LABELS: Record<string, string> = {
  '01': 'Prima Opzione',
  '02': 'Seconda Opzione', 
  '03': 'Terza Opzione',
  '99': 'Altro',
} as const

// Enum values for type safety
export const FEATURE_VALUES = {
  FIRST: '01',
  SECOND: '02',
  THIRD: '03',
  OTHER: '99',
} as const
```

## Requirements

### Italian Labels (MANDATORY)
- ALL user-facing labels must be in Italian
- Use proper Italian terminology
- Follow existing translation patterns

### Type Safety (REQUIRED)
- Always use `as const` for type safety
- Create both labels (for UI) and enum values (for logic)
- Use `Record<string, string>` for label mappings

### Naming Conventions
- Use `_LABELS` suffix for user-facing text
- Use `_VALUES` for enum constants
- Use SCREAMING_SNAKE_CASE for constant names

### SII Specification Alignment
- Match codes to SII specification requirements
- Use numeric string codes as specified (e.g., '01', '02')
- Include comments referencing SII documentation

## Usage in Components

Components must import and use constants like this:

```typescript
import { FEATURE_LABELS } from '@/lib/xml-generator/constants'

// In component render:
{Object.entries(FEATURE_LABELS).map(([value, label]) => (
  <SelectItem key={value} value={value}>
    {label}
  </SelectItem>
))}
```

Never hardcode option labels directly in components.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tate-it) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
