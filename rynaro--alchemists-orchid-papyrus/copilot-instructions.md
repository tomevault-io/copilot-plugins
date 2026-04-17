## alchemists-orchid-papyrus

> **Alchemist's Orchid Papyrus** is a W3C DTCG-compliant design token system featuring a pastel purple-pink palette optimized for accessibility and cross-platform implementation.

# Alchemist's Orchid - Windsurf Rules

## Project Overview

**Alchemist's Orchid Papyrus** is a W3C DTCG-compliant design token system featuring a pastel purple-pink palette optimized for accessibility and cross-platform implementation.

## Core Constraints

### Token Hierarchy (MUST follow)

```
PRIMITIVE → SEMANTIC → COMPONENT
    ↓           ↓           ↓
   Raw        Purpose      UI
  values     mappings    specific
```

Application code may ONLY reference semantic or component tokens. Primitive tokens are internal implementation details.

### Accessibility (MUST meet)

- Normal text: ≥4.5:1 contrast ratio (WCAG AA)
- Large text: ≥3:1 contrast ratio
- UI components: ≥3:1 contrast ratio
- Focus indicators: Required on all interactive elements

## Palette Reference

### Primary Colors
| Token | Value | Usage |
|-------|-------|-------|
| `color.primary` | #CDB4DB | Main brand, primary actions |
| `color.primary.deep` | #6f4a8e | Emphasis, headers, hover states |
| `color.secondary` | #E5D4F1 | Supporting elements, borders |

### Surface Colors
| Token | Value | Usage |
|-------|-------|-------|
| `color.surface` | #F8F5F2 | Main background |
| `color.surface.raised` | #F0ECE9 | Cards, modals, elevated content |

### Text Colors
| Token | Value | Contrast |
|-------|-------|----------|
| `color.text.primary` | #4A4A4A | 7.8:1 on surface ✓ |
| `color.text.secondary` | #6D6D6D | 4.9:1 on surface ✓ |
| `color.text.tertiary` | #8A8A8A | 3.5:1 (large text only) |

### Accent Colors
| Token | Value | Purpose |
|-------|-------|---------|
| `color.accent.warm` | #F8D1E0 | Highlights, notifications |
| `color.accent.cool` | #B9C9E6 | Information, links |
| `color.accent.nature` | #C8E7D5 | Success, positive feedback |

## Code Patterns

### CSS Implementation
```css
/* Always use custom properties with ao- prefix */
.ao-card {
  background-color: var(--ao-color-surface-raised);
  color: var(--ao-color-text-primary);
  border: 1px solid var(--ao-color-secondary);
  border-radius: var(--ao-radius-medium);
}

.ao-card-title {
  color: var(--ao-color-primary-deep);
  font-weight: 600;
}
```

### JavaScript/TypeScript
```typescript
import { tokens } from '@alchemists-orchid/tokens';

const buttonStyles = {
  backgroundColor: tokens.color.primary,
  color: tokens.color.onPrimary,
  border: 'none',
  padding: `${tokens.spacing.sm} ${tokens.spacing.md}`,
};
```

### React Components
```tsx
import { tokens } from '@alchemists-orchid/tokens';

export const Badge = ({ variant = 'default', children }) => {
  const variantStyles = {
    default: { bg: tokens.color.secondary, text: tokens.color.text.primary },
    success: { bg: tokens.color.accent.nature, text: tokens.color.text.primary },
    info: { bg: tokens.color.accent.cool, text: tokens.color.text.primary },
    warm: { bg: tokens.color.accent.warm, text: tokens.color.text.primary },
  };
  
  const style = variantStyles[variant];
  
  return (
    <span style={{ backgroundColor: style.bg, color: style.text }}>
      {children}
    </span>
  );
};
```

## Cascade Memory

### Key Facts to Remember
- Primary purple (#CDB4DB) represents creativity and transformation
- Deep purple (#6f4a8e) provides authority and emphasis
- All text/surface combinations are pre-validated for accessibility
- The `ao-` prefix is used for all CSS custom properties
- Never hardcode hex values in application code

### File Locations
- Token definitions: `tokens/*.json`
- CSS output: `dist/css/variables.css`
- JS output: `dist/js/tokens.js`
- TypeScript types: `dist/types/tokens.d.ts`

## Validation

Before completing tasks:

```bash
# Validate token integrity
npm run validate

# Check accessibility
npm run test:a11y

# Build outputs
npm run build:tokens
```

## What NOT to Do

1. ❌ `color: #CDB4DB;` - Hardcoded hex values
2. ❌ `import { orchid500 }` - Using primitive tokens
3. ❌ Missing focus states on buttons/links
4. ❌ Text contrast below 4.5:1
5. ❌ Using color alone to convey meaning

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Rynaro) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
