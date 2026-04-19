## personalwebsite

> Tailwind CSS usage patterns and responsive design conventions for styling components


# Tailwind CSS Styling Standards

Use Tailwind CSS utility classes for all styling. Follow these patterns for consistent, maintainable styles.

## Basic Usage

- Use Tailwind utility classes in the `className` prop (not `class`)
- No inline styles unless absolutely necessary
- No CSS modules or styled-components
- All styling should be done with Tailwind utility classes

Example from `@app/page.tsx`:
```tsx
<div className="min-h-screen bg-gradient-to-b from-slate-900 via-blue-900 to-slate-900">
```

## Responsive Design Pattern

Follow the mobile-first responsive pattern: base → `sm:` → `md:` → `lg:`

```tsx
// Base styles apply to mobile, then override for larger screens
<div className="px-4 sm:px-6 lg:px-8 py-12 sm:py-16 lg:py-20">
```

Reference responsive patterns in:
- `@app/page.tsx` - responsive padding, text sizes, and spacing
- `@app/components/Navigation.tsx` - responsive navigation layout

## Color Scheme

Use the consistent slate/blue theme throughout the application:

- **Backgrounds**: `slate-900`, `slate-800`, `slate-700`
- **Text**: `slate-100`, `slate-300`, `slate-400`
- **Accents**: `blue-900`, `blue-800`, `blue-600`, `blue-500`, `blue-300`
- **Gradients**: `from-slate-900 via-blue-900 to-slate-900`

Example from `@app/page.tsx`:
```tsx
className="text-xl sm:text-2xl text-blue-300 font-medium"
className="bg-slate-800/90 backdrop-blur-sm rounded-lg shadow-xl border border-blue-800/30"
```

## Common Patterns

### Containers
- Use `mx-auto` for centered containers
- Use `max-w-4xl` or `max-w-7xl` for content width
- Combine with responsive padding: `px-4 sm:px-6 lg:px-8`

### Cards/Sections
```tsx
className="bg-slate-800/90 backdrop-blur-sm rounded-lg shadow-xl border border-blue-800/30 p-8 sm:p-10"
```

### Buttons/Links
```tsx
className="inline-flex items-center px-6 py-3 bg-blue-600 text-white rounded-lg font-medium hover:bg-blue-500 transition-colors shadow-lg hover:shadow-blue-500/50 hover:scale-105"
```

### Typography
- Headings: `text-4xl sm:text-5xl lg:text-6xl font-bold text-slate-100`
- Body text: `text-slate-300 leading-relaxed`
- Subheadings: `text-2xl font-semibold text-blue-300`

## Utility Classes Reference

### Spacing
- Padding: `p-4`, `px-6`, `py-8`, `pt-12` (with responsive: `sm:p-6`, `lg:py-20`)
- Margin: `mb-4`, `mt-8`, `mx-auto`, `gap-3`, `gap-4`

### Layout
- Flexbox: `flex`, `flex-wrap`, `items-center`, `justify-between`, `justify-center`
- Grid: Use Tailwind grid utilities when needed
- Positioning: `sticky`, `top-0`, `z-50`

### Visual Effects
- Shadows: `shadow-lg`, `shadow-xl`
- Borders: `border`, `border-t`, `border-blue-800/30`, `border-blue-800/50`
- Backdrop: `backdrop-blur-sm`
- Opacity: `/90`, `/50`, `/30` (e.g., `bg-slate-800/90`)
- Transitions: `transition-colors`, `hover:bg-blue-500`, `hover:scale-105`

### Text Styling
- Font weights: `font-medium`, `font-semibold`, `font-bold`
- Text sizes: `text-sm`, `text-xl`, `text-2xl`, `text-4xl` (with responsive variants)
- Line height: `leading-relaxed`

## Best Practices

- Group related classes together (spacing, colors, layout, effects)
- Use responsive variants consistently (base → sm → md → lg)
- Maintain consistent spacing scale (4, 6, 8, 10, 12, 16, 20)
- Use opacity variants (`/90`, `/50`) for overlays and borders
- Add transitions for interactive elements
- Use semantic color names from the Tailwind palette

## Configuration

Tailwind configuration is in `@tailwind.config.ts`. The content paths are:
- `./pages/**/*.{js,ts,jsx,tsx,mdx}`
- `./components/**/*.{js,ts,jsx,tsx,mdx}`
- `./app/**/*.{js,ts,jsx,tsx,mdx}`

## References

- Styling examples: `@app/page.tsx`, `@app/components/Navigation.tsx`
- Tailwind config: `@tailwind.config.ts`
- Global styles: `@app/globals.css`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/haimlt1995) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
