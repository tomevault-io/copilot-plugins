## ianarazny-github-io

> - **Background**: `#0f172a` - Dark navy background

# Portfolio Styling Reference

## Color Palette

### Dark Navy Theme (Default - Brittany Chiang style)
- **Background**: `#0f172a` - Dark navy background
- **Foreground**: `#e2e8f0` - Primary text color (slate-200)
- **Accent**: `#5eead4` - Teal accent for highlights and CTAs
- **Accent Hover**: `#2dd4bf` - Brighter teal for hover states
- **Card Background**: `#1e293b` - Slightly lighter navy for cards (slate-800)
- **Card Border**: `#334155` - Subtle borders (slate-700)
- **Text Muted**: `#94a3b8` - Secondary/muted text (slate-400)
- **Text Highlight**: `#e2e8f0` - Bold/highlighted text (slate-200)

### Light Theme (Alternative)
- **Background**: `#ffffff`
- **Foreground**: `#171717`

## Typography

### Font Family
- **Primary**: 'Inter' - Modern, clean sans-serif
- **Weights Available**: 300, 400, 500, 600, 700

### Heading Sizes
- **h1**: 3.5rem (56px) - Main page titles, reduces to 2.5rem on mobile
- **h2**: 2.5rem (40px) - Section headings, reduces to 2rem on mobile
- **h3**: 2rem (32px) - Subsection headings, reduces to 1.5rem on mobile
- **h4**: 1.5rem (24px) - Card titles and smaller headings

### Usage Guidelines
- **Headings**: Use weights 600-700 for emphasis
- **Body Text**: Use weight 400 for readability
- **Labels/Captions**: Use weight 300-400

## CSS Variables

Use these CSS custom properties throughout the project:

```css
var(--background)      /* Main background */
var(--foreground)      /* Main text color */
var(--accent)          /* Accent/brand color */
var(--accent-hover)    /* Hover states */
var(--card-bg)         /* Card backgrounds */
var(--card-border)     /* Border colors */
var(--text-muted)      /* Secondary text */
```

## Tailwind Configuration

The project uses Tailwind CSS with the standard directives:
- `@tailwind base` - Base styles and resets
- `@tailwind components` - Component classes
- `@tailwind utilities` - Utility classes

Custom theme values are defined inline using `@theme inline`.

**Note**: If using VSCode, ensure the Tailwind CSS IntelliSense extension is installed for autocomplete and linting support.

## Design Patterns

### Sections
- **Padding**: 5rem vertical, 2rem horizontal (3rem/1.5rem on mobile)
- **Max Width**: 1200px centered
- **Spacing**: Use consistent margins between elements
- **Headings**: Gradient text effect from foreground to accent color
- **Paragraphs**: Muted color, centered, max-width 800px for readability

### Cards
- Background: `var(--card-bg)`
- Border: `1px solid var(--card-border)`
- Add subtle hover effects with `var(--accent)`

### Tech Stack Items
- Background: `var(--card-bg)`
- Border: `1px solid var(--card-border)`
- Hover: Border becomes accent color with subtle background tint
- Transition: Smooth 300ms with slight upward transform
- Layout: Flexbox with wrap, centered, 1rem gap

### Buttons/Links
- Primary: Use `var(--accent)` background
- Hover: Transition to `var(--accent-hover)`
- Add smooth transitions (200-300ms)

### Sections
- Use consistent spacing (padding/margin)
- Maintain visual hierarchy with typography weights
- Consider glassmorphism effects for modern look

## Browser Features

### Smooth Scrolling
- Enabled globally via `scroll-behavior: smooth`

### Custom Scrollbar
- Width: 8px
- Track: Matches background
- Thumb: Uses `var(--card-border)` with 4px border-radius

### Selection Styling
- Background: `var(--accent)` (blue)
- Text color: white

## Best Practices

1. **Consistency**: Always use CSS variables for colors
2. **Accessibility**: Maintain sufficient color contrast ratios
3. **Responsiveness**: Design mobile-first, enhance for larger screens
4. **Performance**: Leverage Tailwind utilities for common patterns
5. **Dark Mode**: Primary theme is dark; ensure all components work well

## Component Examples

### Section Container
```jsx
<section className="min-h-screen px-6 py-20">
<section>
  <h2>About Me</h2>
  <p>Your introduction text here...</p>
  
  <h3>Tech Stack</h3>
  <div className="tech-stack">
    <span className="tech-item">TypeScript</span>
    <span className="tech-item">React</span>
    <span className="tech-item">Next.js</span>
  </div>
</section>
```

### Card Component
```jsx
<div className="p-6 rounded-lg border border-[var(--card-border)] bg-[var(--card-bg)] hover:border-[var(--accent)] transition-colors">
  {/* Card content */}
</div>
```

### Text Styles
```jsx
<h1 className="text-4xl font-bold">Main Heading</h1>
<h1>Main Page Title (56px)</h1>
<h2>Section Heading (40px)</h2>
<h3>Subsection (32px)</h3>
<p className="text-[var(--text-muted)]">Secondary text</p>
```

## Notes

- The project uses Next.js with App Router
- Inter font is loaded from Google Fonts
- Dark theme is the default experience
- Prefers-color-scheme media query is available for automatic theme switching

## Troubleshooting

### Unknown at rule @tailwind warning

If you see `Unknown at rule @tailwindcss(unknownAtRules)` warning in VSCode:

1. Create a `.vscode/settings.json` file in the project root
2. Add: `{ "css.lint.unknownAtRules": "ignore" }`
3. Alternatively, install the "Tailwind CSS IntelliSense" extension for better support

This is a linting issue and doesn't affect functionality.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ianarazny)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ianarazny)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
