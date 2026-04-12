## swiftware-client-site

> Design system, theming, and styling patterns for SwiftWare


# SwiftWare Design System & Theming

## Color Palette

### Primary Colors

```css
/* Blue Primary */
--blue-500: rgb(59, 130, 246);
--blue-600: rgb(37, 99, 235);

/* Purple Primary */
--purple-500: rgb(168, 85, 247);
--purple-600: rgb(147, 51, 234);
```

### Background Colors

```css
/* Base backgrounds */
--zinc-950: rgb(9, 9, 11); /* Primary background */
--zinc-900: rgb(24, 24, 27); /* Secondary background */
--zinc-800: rgb(39, 39, 42); /* Elevated surfaces */
```

### Text Colors

```css
--zinc-100: rgb(244, 244, 245); /* Primary text */
--zinc-200: rgb(228, 228, 231); /* Secondary text */
--zinc-300: rgb(212, 212, 216); /* Tertiary text */
--zinc-400: rgb(161, 161, 170); /* Muted text */
```

## Gradient Patterns

### Primary Gradients

```css
/* Hero gradient */
.hero-gradient {
  background: linear-gradient(135deg, var(--blue-500), var(--purple-600));
}

/* Text gradient */
.text-gradient {
  background: linear-gradient(135deg, var(--blue-500), var(--purple-500));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

/* Button gradient */
.btn-gradient {
  background: linear-gradient(135deg, var(--blue-500), var(--purple-500));
}
```

### Opacity Levels

```css
/* Card backgrounds */
--card-bg-primary: rgba(255, 255, 255, 0.04);
--card-bg-secondary: rgba(255, 255, 255, 0.08);

/* Borders */
--border-primary: rgba(255, 255, 255, 0.1);
--border-secondary: rgba(255, 255, 255, 0.2);

/* Glows */
--glow-primary: rgba(59, 130, 246, 0.05);
--glow-secondary: rgba(168, 85, 247, 0.08);
```

## Tailwind Configuration

### Custom Colors (tailwind.config.js)

```javascript
module.exports = {
  theme: {
    extend: {
      colors: {
        brand: {
          blue: {
            500: 'rgb(59, 130, 246)',
            600: 'rgb(37, 99, 235)',
          },
          purple: {
            500: 'rgb(168, 85, 247)',
            600: 'rgb(147, 51, 234)',
          },
        },
      },
      backgroundImage: {
        'hero-gradient':
          'linear-gradient(135deg, rgb(59, 130, 246), rgb(147, 51, 234))',
        'text-gradient':
          'linear-gradient(135deg, rgb(59, 130, 246), rgb(168, 85, 247))',
      },
    },
  },
};
```

### Utility Classes

```css
/* Glow effects */
.shadow-brand-glow {
  box-shadow: 0 0 20px rgba(59, 130, 246, 0.3);
}

/* Animated gradients */
.bg-animated-gradient {
  background: linear-gradient(
    45deg,
    rgba(59, 130, 246, 0.1),
    rgba(168, 85, 247, 0.1),
    rgba(59, 130, 246, 0.1)
  );
  background-size: 400% 400%;
  animation: gradient-shift 8s ease infinite;
}

@keyframes gradient-shift {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}
```

## Component Styling Patterns

### Card Components

```typescript
<div className="
  bg-white/[0.04] border border-white/10
  hover:bg-white/[0.08] hover:border-blue-500/50
  transition-all duration-300
  rounded-xl p-6 backdrop-blur-sm
">
  {/* Card content */}
</div>
```

### Button Components

```typescript
<button className="
  bg-gradient-to-r from-blue-500 to-purple-500
  hover:shadow-[0_0_20px_rgba(59,130,246,0.3)]
  text-white font-medium
  px-6 py-3 rounded-lg
  transition-all duration-200
  focus:ring-2 focus:ring-blue-400/70 focus:outline-none
">
  Button Text
</button>
```

### Text Styling

```typescript
<h1 className="
  bg-gradient-to-r from-white via-blue-100 to-purple-100
  bg-clip-text text-transparent
  text-4xl md:text-6xl font-bold
  leading-tight
">
  Headline Text
</h1>

<p className="text-zinc-300 leading-relaxed">
  Body text content
</p>
```

## Animation Patterns

### Motion/React Variants

```typescript
const containerVariants = {
  hidden: { opacity: 0 },
  visible: {
    opacity: 1,
    transition: {
      staggerChildren: 0.1,
      delayChildren: 0.2,
    },
  },
};

const itemVariants = {
  hidden: { y: 20, opacity: 0 },
  visible: {
    y: 0,
    opacity: 1,
    transition: {
      type: 'spring',
      stiffness: 100,
      damping: 12,
    },
  },
};
```

### CSS Transitions

```css
/* Smooth transitions */
.transition-all {
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.transition-colors {
  transition:
    color 0.2s ease,
    background-color 0.2s ease,
    border-color 0.2s ease;
}

/* Focus states */
.focus-ring {
  transition: box-shadow 0.2s ease;
}

.focus-ring:focus {
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.5);
}
```

## Responsive Design

### Breakpoint System

```typescript
// Mobile-first approach
<div className="
  grid grid-cols-1
  md:grid-cols-2
  lg:grid-cols-3
  xl:grid-cols-4
  gap-4 md:gap-6 lg:gap-8
">
  {/* Content */}
</div>

// Container patterns
<div className="
  max-w-7xl mx-auto
  px-4 sm:px-6 lg:px-8
  py-8 md:py-12 lg:py-16
">
  {/* Content */}
</div>
```

### Mobile-Specific Patterns

```typescript
// Touch-friendly interactions
<button className="
  min-h-[44px] min-w-[44px] /* iOS touch target size */
  px-4 py-2
  active:scale-95 /* Touch feedback */
  transition-transform duration-75
">
  Touch Button
</button>

// Responsive text scaling
<h1 className="
  text-2xl sm:text-3xl md:text-4xl lg:text-5xl xl:text-6xl
  leading-tight
">
  Responsive Headline
</h1>
```

## Dark Theme Optimization

### High Contrast Elements

```typescript
// High contrast cards
<div className="
  bg-zinc-900/80 backdrop-blur-md
  border border-zinc-700/50
  hover:border-zinc-600/70
  shadow-xl
">
  {/* Content */}
</div>

// Elevated content
<div className="
  bg-zinc-800/90 backdrop-blur-lg
  border border-zinc-600/30
  shadow-2xl
">
  {/* Important content */}
</div>
```

### Accessibility Colors

```typescript
// Focus indicators
.focus-visible {
  outline: 2px solid rgb(59, 130, 246);
  outline-offset: 2px;
}

// Error states
.error-text {
  color: rgb(239, 68, 68); /* red-500 */
}

.error-border {
  border-color: rgb(239, 68, 68);
}
```

## Performance Considerations

### CSS Optimization

```css
/* Use CSS custom properties for theming */
:root {
  --color-primary: rgb(59, 130, 246);
  --color-secondary: rgb(168, 85, 247);
  --bg-primary: rgb(9, 9, 11);
}

/* Minimize repaints with transform instead of layout properties */
.animate-slide-in {
  transform: translateX(-100%);
  transition: transform 0.3s ease;
}

.animate-slide-in.visible {
  transform: translateX(0);
}
```

### Image Optimization

```typescript
// Next.js Image component with proper sizing
import Image from "next/image";

<Image
  src="/hero-image.jpg"
  alt="Hero image"
  width={1200}
  height={600}
  className="w-full h-auto"
  priority // For above-the-fold images
  placeholder="blur" // With blur placeholder
/>
```

## CSS Architecture

### Global Styles (global.css)

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    /* CSS custom properties for theming */
    --color-bg-primary: rgb(9, 9, 11);
    --color-text-primary: rgb(244, 244, 245);
    --color-accent-blue: rgb(59, 130, 246);
    --color-accent-purple: rgb(168, 85, 247);
  }

  html {
    scroll-behavior: smooth;
  }

  body {
    @apply bg-zinc-950 text-zinc-100 antialiased;
    font-feature-settings:
      'rlig' 1,
      'calt' 1;
  }
}

@layer components {
  .btn-primary {
    @apply bg-gradient-to-r from-blue-500 to-purple-500
           hover:shadow-[0_0_20px_rgba(59,130,246,0.3)]
           text-white font-medium px-6 py-3 rounded-lg
           transition-all duration-200;
  }

  .card-base {
    @apply bg-white/[0.04] border border-white/10
           rounded-xl backdrop-blur-sm;
  }
}

@layer utilities {
  .text-balance {
    text-wrap: balance;
  }
}
```

## Component-Specific Styles

```typescript
// Use CSS modules or styled-components for complex animations
import styles from './Component.module.css';

<div className={styles.animatedElement}>
  {/* Content */}
</div>
```

```css
/* Component.module.css */
.animatedElement {
  animation: fadeInUp 0.6s ease-out;
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
```

## Testing Visual Consistency

### Visual Regression Testing

```typescript
// Example test for visual consistency
describe("Button Component", () => {
  it("matches visual snapshot", () => {
    const { container } = render(<Button>Click me</Button>);
    expect(container.firstChild).toMatchSnapshot();
  });
});
```

### Theme Testing

```typescript
// Test different theme states
describe('Theming', () => {
  it('applies correct colors in light mode', () => {
    // Test implementation
  });

  it('maintains contrast ratios', () => {
    // Accessibility testing
  });
});
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SwiftWareCo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/SwiftWareCo)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
