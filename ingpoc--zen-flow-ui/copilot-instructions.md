## zen-flow-design-system

> Zen Flow UI follows Japanese design principles combined with modern web aesthetics:

# Zen Flow UI Design System

## Philosophy & Principles

Zen Flow UI follows Japanese design principles combined with modern web aesthetics:

### Core Design Principles
- **Ma (間)**: Purposeful negative space and breathing room
- **Kanso (簡素)**: Simplicity and elimination of clutter  
- **Kokō (考)**: Subtle elegance through restraint
- **Wabi-Sabi**: Beauty in imperfection and impermanence

### Design Filters
Every design decision must pass through:
1. **Function**: Does it serve the user's goal?
2. **Form**: Is it visually coherent?
3. **Delight**: Does it spark joy?

## Color Palette

### Neutral Scale (Primary)
```css
--zen-void: #0a0a0a;     /* Deepest black, for text and strong contrast */
--zen-ink: #1a1a1a;      /* Primary text, dark surfaces */
--zen-shadow: #2a2a2a;   /* Subtle borders, dividers */
--zen-stone: #4a4a4a;    /* Secondary text, disabled states */
--zen-mist: #8a8a8a;     /* Placeholder text, subtle elements */
--zen-cloud: #dadada;    /* Light borders, hover states */
--zen-paper: #fafafa;    /* Light backgrounds, cards */
--zen-light: #ffffff;    /* Pure white, primary backgrounds */
```

### Accent Colors (Semantic)
```css
--zen-accent: #ff4757;   /* Primary brand, CTAs, errors */
--zen-water: #3742fa;    /* Links, information, primary actions */
--zen-leaf: #26de81;     /* Success states, confirmations */
--zen-sun: #fed330;      /* Warnings, highlights */
```

### Usage Guidelines
- Use neutral scale for 90% of interface elements
- Reserve accent colors for semantic meaning and CTAs
- Never use more than 2 accent colors in a single view
- Maintain 4.5:1 contrast ratio minimum for accessibility

## Spacing System

### Base Scale (8px grid)
```css
--zen-space-xs: 4px;   /* 0.25rem - Tight spacing */
--zen-space-sm: 8px;   /* 0.5rem - Small gaps */
--zen-space-md: 16px;  /* 1rem - Standard spacing */
--zen-space-lg: 24px;  /* 1.5rem - Section spacing */
--zen-space-xl: 32px;  /* 2rem - Large gaps */
--zen-space-2xl: 48px; /* 3rem - Major sections */
--zen-space-3xl: 64px; /* 4rem - Page-level spacing */
```

### Component-Specific Spacing
```css
--zen-padding-input: 12px 16px;        /* Form inputs */
--zen-padding-button: 8px 16px;        /* Buttons */
--zen-padding-card: 24px;              /* Card content */
--zen-margin-stack: 16px;              /* Vertical rhythm */
--zen-gap-form: 12px;                  /* Form field gaps */
```

### Responsive Spacing
- Mobile: Use smaller spacing values (multiply by 0.75)
- Desktop: Use standard values
- Large screens: Use larger values (multiply by 1.25)

## Typography Scale

### Font Weights
```css
--zen-weight-light: 300;
--zen-weight-normal: 400;
--zen-weight-medium: 500;
--zen-weight-semibold: 600;
--zen-weight-bold: 700;
```

### Font Sizes
```css
--zen-text-xs: 0.75rem;    /* 12px - Small labels */
--zen-text-sm: 0.875rem;   /* 14px - Body text, captions */
--zen-text-base: 1rem;     /* 16px - Primary body text */
--zen-text-lg: 1.125rem;   /* 18px - Large body text */
--zen-text-xl: 1.25rem;    /* 20px - Small headings */
--zen-text-2xl: 1.5rem;    /* 24px - Section headings */
--zen-text-3xl: 1.875rem;  /* 30px - Page headings */
--zen-text-4xl: 2.25rem;   /* 36px - Hero headings */
```

### Line Heights
```css
--zen-leading-tight: 1.25;   /* Headings */
--zen-leading-normal: 1.5;   /* Body text */
--zen-leading-relaxed: 1.75; /* Large text blocks */
```

## Border & Radius

### Border Widths
```css
--zen-border-thin: 1px;    /* Subtle dividers */
--zen-border-normal: 2px;  /* Standard borders */
--zen-border-thick: 4px;   /* Emphasis borders */
```

### Border Radius
```css
--zen-radius-sm: 4px;   /* Small elements, tags */
--zen-radius-md: 8px;   /* Buttons, inputs */
--zen-radius-lg: 12px;  /* Cards, modals */
--zen-radius-xl: 16px;  /* Large containers */
--zen-radius-full: 9999px; /* Pills, circular */
```

## Shadows & Elevation

### Shadow Levels
```css
--zen-shadow-sm: 0 1px 2px 0 rgb(0 0 0 / 0.05);
--zen-shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.1);
--zen-shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.1);
--zen-shadow-xl: 0 20px 25px -5px rgb(0 0 0 / 0.1);
```

### Usage Guidelines
- sm: Subtle elevation (buttons, inputs)
- md: Standard elevation (cards, dropdowns)
- lg: Modal elevation (overlays, tooltips)
- xl: High elevation (notifications, alerts)

## Motion & Animation

### Duration Scale
```css
--zen-duration-fast: 150ms;    /* Micro-interactions */
--zen-duration-normal: 300ms;  /* Standard transitions */
--zen-duration-slow: 500ms;    /* Complex animations */
```

### Easing Functions
```css
--zen-ease-out: cubic-bezier(0.25, 0.46, 0.45, 0.94);
--zen-ease-in-out: cubic-bezier(0.645, 0.045, 0.355, 1);
--zen-ease-bounce: cubic-bezier(0.68, -0.55, 0.265, 1.55);
```

### Animation Principles
- Respect `prefers-reduced-motion`
- Use purposeful motion that guides attention
- Maintain consistent timing across similar elements
- Favor CSS transitions over JavaScript animations for simple effects

## Component Guidelines

### Interactive States
All interactive elements must have:
- **Default**: Base appearance
- **Hover**: Subtle elevation or color change
- **Focus**: Clear focus indicator (2px outline)
- **Active**: Pressed/clicked state
- **Disabled**: Reduced opacity with cursor: not-allowed

### Content Guidelines
- Use sentence case for UI labels
- Keep button text concise (1-3 words ideal)
- Provide clear error messages
- Use progressive disclosure for complex forms

### Accessibility Requirements
- Minimum 44px touch targets on mobile
- Color contrast ratio ≥ 4.5:1 for text
- Focus indicators must be visible
- Screen reader compatible markup
- Keyboard navigation support

## Implementation Notes

Reference design tokens in components using CSS custom properties:
```css
.zen-button {
  padding: var(--zen-padding-button);
  border-radius: var(--zen-radius-md);
  transition: all var(--zen-duration-normal) var(--zen-ease-out);
}
```

Never use hardcoded values - always reference the design system.

---
> Source: [ingpoc/zen-flow-ui](https://github.com/ingpoc/zen-flow-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
