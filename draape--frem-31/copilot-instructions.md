## frem-31

> - TypeScript (strict mode) with Next.js 16+ App Router + Next.js, React 19+, Panda CSS, Arc UI

# web Development Guidelines

## Active Technologies

- TypeScript (strict mode) with Next.js 16+ App Router + Next.js, React 19+, Panda CSS, Arc UI

## Project Structure

```text
src/
tests/
```

## Commands

npm test && npm run lint

<!-- MANUAL ADDITIONS START -->

## Panda CSS Styling Patterns

### Component Styling Approach

Choose the right Panda CSS function based on component complexity:

1. **sva() - Multi-part components**
   - Use for components with multiple styled elements (slots)
   - Define styles at the **bottom of file** after component export
   - Use short variable names: `s`, `style`, or `styles`
   - Examples: Hero, PageHeader, Button

2. **cva() - Single element with variants**
   - Use for components with one element but multiple variants
   - Example: Section component with maxWidth/padding/background variants

3. **css() - Simple components**
   - Use for single elements without variants
   - Example: Badge component

### Code Organization

**Component file structure:**

```typescript
// 1. Imports
import { sva } from '@/styled-system/css';

// 2. Types
type ComponentProps = { ... };

// 3. Component (exported, at top)
export function Component({ ... }: ComponentProps) {
  const s = styles();
  return <div className={s.root}>...</div>;
}

// 4. Styles (at bottom)
const styles = sva({
  slots: ['root', 'title'],
  base: { ... },
});
```

### Property Shorthand

Prefer shorthand properties in Panda CSS:

- `bg` not `backgroundColor`
- `w` not `width`
- `pos` not `position`
- `flexDir` not `flexDirection`
- `maxW` not `maxWidth`

### Section Component Pattern

Use Section component for max-width containers instead of duplicating container styles:

```typescript
<Section maxWidth="content|layout" padding="none|sm|md|lg" background="transparent|white|blue|yellow">
  {/* content */}
</Section>
```

**When to use:**

- `maxWidth="content"` (800px) - for content-focused sections (articles, forms)
- `maxWidth="layout"` (1280px) - for full-width layouts (headers, footers, hero)
- `padding="none|sm|md|lg"` - standardized vertical padding (0, 2rem, 3rem, 4rem)
- `background` - standardized background colors

### Design Tokens

Always use design tokens instead of hardcoded values:

**Spacing tokens:**

- Use: `'4'`, `'6'`, `'8'`, `'12'`, `'16'`
- Never: `'1rem'`, `'24px'`, `'2.5rem'`

**Color tokens:**

- Use: `'blue.800'`, `'yellow.400'`, `'grey.950'`
- Never: `'#HEX'`, `'rgb()'`, `'rgba()'`

**Breakpoints:**

- Use: `md: {...}`, `sm: {...}`, `lg: {...}`
- Never: `'@media (min-width: 768px)'`

### panda.config.ts Guidelines

Keep panda.config.ts focused on design tokens only:

**✅ What belongs:**

- Design tokens (colors, fonts, spacing, breakpoints)
- textStyles for typography system
- Global configuration

**❌ What does NOT belong:**

- Component-specific styles (use sva/cva/css in component files)
- Slot recipes (co-locate styling with components)

<!-- MANUAL ADDITIONS END -->

---
> Source: [draape/Frem-31](https://github.com/draape/Frem-31) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
