## waku

> Tailwind CSS styling rules, class extraction, and semantic naming conventions


# Tailwind CSS and Styling Rules

## Core Principles

- Tailwind CSS v4 is mandatory
- Long utility chains are NOT allowed in JSX
- Reusable styles must live in global.css
- Styling must be semantic, not visual-only

## Global.css Rules

- Use `@layer base, components, utilities`
- Define:
  • Colors
  • Surfaces
  • Typography
  • Common layouts
- Add comments for each section

### Example

```css
@layer components {
  /* Cards */
  .card-surface {
    @apply rounded-xl bg-surface p-4 shadow-sm;
  }
}
```

## JSX Styling Rules

- JSX should use:
  • Semantic class names
  • Minimal utilities
- If className exceeds ~6 utilities:
  → Extract to global.css

### Bad Example

```typescript
<div className="flex items-center justify-between rounded-xl bg-white px-4 py-3 shadow-sm">
```

### Good Example

```typescript
<div className="card-surface card-row">
```

## Responsiveness

- Mobile-first by default
- Use Tailwind breakpoints intentionally
- Avoid hardcoded widths/heights unless required

## Enforcement

- If reusable styling is detected → extract it
- If global.css is missing styles → suggest additions
- Do NOT invent colors or spacing
- Always reference existing design tokens

If unsure, request:
- `[styles/global.css]` or `[app/globals.css]`

## Class Extraction Rules

### When to Extract to globals.css

Extract class patterns to `globals.css` if:
1. **Reusability**: Pattern appears 2+ times across components
2. **Length**: Class chain has >5 utility classes
3. **Semantic meaning**: Represents a distinct UI element (card, button, input, etc.)

### Extraction Pattern

```css
/* In globals.css */
@layer components {
  .card-surface {
    @apply rounded-xl border border-accent-200/30 bg-white p-4 shadow-sm;
  }

  .text-muted {
    @apply text-accent-200;
  }

  .button-primary {
    @apply inline-flex items-center justify-center rounded-full border border-accent-200/60 bg-white text-background-dark transition hover:border-background-dark hover:bg-accent-900;
  }
}
```

```typescript
// In component
<div className="card-surface">
  <p className="text-muted">Content</p>
  <button className="button-primary">Click</button>
</div>
```

## Semantic Class Naming

### Good Names (Semantic)
- ✅ `.card-surface` - describes what it is
- ✅ `.text-muted` - describes purpose
- ✅ `.button-primary` - describes role
- ✅ `.panel-header` - describes location/role
- ✅ `.upload-area` - describes function

### Bad Names (Non-semantic)
- ❌ `.style1`, `.style2` - meaningless
- ❌ `.red-box` - describes appearance, not purpose
- ❌ `.big-text` - describes size, not meaning
- ❌ `.rounded-border` - describes style, not function

## Common Patterns to Extract

### Card/Panel Pattern
```css
@layer components {
  .card-surface {
    @apply rounded-xl border border-accent-200/30 bg-white p-4 shadow-sm;
  }
}
```

### Button Pattern
```css
@layer components {
  .button-icon {
    @apply inline-flex items-center justify-center w-8 h-8 rounded-full border border-accent-200/60 bg-white text-background-dark transition hover:border-background-dark hover:bg-accent-900;
  }
}
```

### Text Patterns
```css
@layer components {
  .text-muted {
    @apply text-accent-200;
  }

  .text-heading {
    @apply text-base font-semibold text-background-dark;
  }
}
```

### Layout Patterns
```css
@layer components {
  .flex-center {
    @apply flex items-center justify-center;
  }

  .flex-between {
    @apply flex items-center justify-between;
  }
}
```

## Inline Class Limits

### Acceptable Inline (≤6 classes)
```typescript
// ✅ OK - short, specific
<div className="flex items-center gap-2">
<div className="text-xl font-semibold">
```

### Must Extract (>6 classes or repeated)
```typescript
// ❌ TOO LONG - extract to globals.css
<header className="w-full flex items-center justify-between rounded-full border border-accent-200/30 bg-white px-10 py-3 shadow-sm">

// ✅ EXTRACTED
<header className="header-container">
```

## Current Violations

Review these components for extraction:
- `app-header.tsx` - long class chains on header and buttons
- `base-panel.tsx` - panel styling pattern (reusable)
- `upload-feature.tsx` - upload area styling (reusable)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dhanushdotcodes) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
