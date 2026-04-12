## fit-report

> The application uses DaisyUI for theming with the following available themes:

# FitReport Design Guidelines

## Framework Stack
- Next.js 14
- Tailwind CSS
- DaisyUI
- TypeScript

## Theme Configuration

### DaisyUI Themes
The application uses DaisyUI for theming with the following available themes:
- `dark` (default dark theme)

### Base Colors
DaisyUI provides semantic color names that should be used consistently:
- `primary` - Main brand color
- `secondary` - Secondary brand color
- `accent` - Accent color for highlights
- `neutral` - Neutral color for text and backgrounds
- `base-100` - Base background color
- `base-200` - Slightly darker background color
- `base-content` - Base text color

## Component Classes

### Buttons
```html
<!-- Primary Button -->
<button className="btn btn-primary">Button</button>

<!-- Secondary Button -->
<button className="btn btn-secondary">Button</button>

<!-- Wide Button -->
<button className="btn btn-primary btn-wide">Button</button>
```

### Cards
```html
<div className="card bg-base-100 shadow-xl">
  <div className="card-body">
    <!-- Card content -->
  </div>
</div>
```

### Form Elements
```html
<!-- Form Control Container -->
<div className="form-control w-full">
  <!-- Label -->
  <label className="label">
    <span className="label-text">Label</span>
  </label>
  
  <!-- Input -->
  <input className="input input-bordered w-full" />
</div>
```

### Text Styles
- Headers: `text-3xl font-bold`
- Subheaders: `text-xl font-semibold`
- Body Text: `text-base-content`
- Muted Text: `text-base-content/80`

### Layout
- Container: `container mx-auto px-8`
- Section Padding: `p-8`
- Flex Layout: `flex items-center justify-between`
- Grid Layout: `grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6`

### Spacing
- Standard margin bottom: `mb-6`
- Standard padding: `p-6`
- Standard gap: `gap-3`

### Responsive Design
- Mobile First Approach
- Breakpoints:
  - `sm`: 640px
  - `md`: 768px
  - `lg`: 1024px
  - `xl`: 1280px

### Animation Classes
```css
animation: {
  opacity: "opacity 0.25s ease-in-out",
  appearFromRight: "appearFromRight 300ms ease-in-out",
  wiggle: "wiggle 1.5s ease-in-out infinite",
  popup: "popup 0.25s ease-in-out",
  shimmer: "shimmer 3s ease-out infinite alternate"
}
```

## Best Practices

### Component Structure
1. Use semantic HTML elements
2. Maintain consistent spacing
3. Follow mobile-first approach
4. Use DaisyUI components when available
5. Implement responsive design patterns

### Color Usage
1. Use semantic color names from DaisyUI
2. Maintain consistent color hierarchy
3. Use opacity modifiers for variations
4. Follow accessibility guidelines for contrast

### Typography
1. Use the Inter font family (default)
2. Maintain consistent text hierarchy
3. Use appropriate font weights
4. Follow responsive text sizing

### Accessibility
1. Include proper ARIA labels
2. Maintain sufficient color contrast
3. Ensure keyboard navigation
4. Provide screen reader support

## File Organization
- Components: `/components`
- Pages: `/app`
- Styles: `/styles`
- Configuration: `/config`
- Types: `/types`

## Version Control
- Follow consistent commit messages
- Document major style changes
- Update design documentation when needed

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rileymanderson19)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rileymanderson19)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
