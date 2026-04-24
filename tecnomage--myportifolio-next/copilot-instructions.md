## myportifolio-next

> This file provides comprehensive guidance to Claude Code (claude.ai/code) when working with code in this repository, emphasizing best practices for code quality, performance optimization, and exceptional user experience.

# CLAUDE.md

This file provides comprehensive guidance to Claude Code (claude.ai/code) when working with code in this repository, emphasizing best practices for code quality, performance optimization, and exceptional user experience.

## Project Overview

This is a high-performance personal portfolio website built with Next.js 15.2.0 and React 19, showcasing full-stack development skills and services. The portfolio features:

- Modern responsive design with CSS modules and mobile-first approach
- Interactive SVG animations and optimized particle systems
- Smooth scrolling navigation with accessibility considerations
- Mobile-friendly responsive layout with touch-optimized interactions
- Contact form with progressive enhancement and social media integration
- Skills categorization with semantic structure (Frontend, Backend, DevOps, CMS)

## Development Commands

- `npm run dev` - Start development server (localhost:3000)
- `npm run build` - Build production version with optimization analysis
- `npm start` - Start production server
- `npm run lint` - Run ESLint for code quality
- `npm run lint:fix` - Auto-fix linting issues
- `npm run type-check` - Run TypeScript type checking
- `npm run analyze` - Bundle analyzer for performance optimization

## Tech Stack

- **Framework**: Next.js 15.2.0 (App Router) with static optimization
- **Frontend**: React 19 with Server Components, TypeScript 5
- **Styling**: CSS Modules with custom animations and CSS-in-JS fallback
- **Icons**: React Icons (FA, SI, TB, HI, MD) with tree-shaking
- **Animations**: Framer Motion with reduced motion support, custom SVG animations
- **CMS**: Sanity (configured but not fully implemented)
- **Performance**: Image optimization, lazy loading, code splitting
- **Accessibility**: ARIA labels, keyboard navigation, screen reader support
- **Linting**: ESLint with Next.js rules, Prettier for formatting

## Architecture & File Structure

```
src/
├── app/
│   ├── components/
│   │   ├── ui/              # Reusable UI components
│   │   ├── sections/        # Page sections (Hero, Services, etc.)
│   │   ├── Footer.tsx       # Social links and contact info
│   │   └── Footer.module.css
│   ├── hooks/               # Custom React hooks
│   ├── utils/               # Utility functions and helpers
│   ├── constants/           # App constants and configuration
│   ├── globals.css          # Global styles and CSS variables
│   ├── page.tsx             # Main portfolio page with all sections
│   ├── page.module.css      # Homepage styles and animations
│   └── layout.tsx           # Root layout with optimized font loading
├── lib/
│   ├── sanity.ts           # Sanity CMS client configuration
│   └── performance.ts      # Performance utilities and monitoring
└── public/
    ├── images/             # Optimized images with multiple formats
    └── icons/              # SVG icons and favicons
```

## 🚀 Performance Best Practices

### Core Web Vitals Optimization
- **LCP (Largest Contentful Paint)**: < 2.5s
  - Optimize hero section images with `next/image`
  - Preload critical resources
  - Use efficient image formats (WebP, AVIF)
  
- **FID (First Input Delay)**: < 100ms
  - Minimize JavaScript execution time
  - Use React 19 concurrent features
  - Implement proper code splitting

- **CLS (Cumulative Layout Shift)**: < 0.1
  - Define image dimensions explicitly
  - Reserve space for dynamic content
  - Use CSS containment where appropriate

### Implementation Guidelines
```typescript
// Example: Optimized image component
import Image from 'next/image'

const OptimizedHero = () => (
  <Image
    src="/hero-image.jpg"
    alt="Portfolio hero"
    width={1200}
    height={600}
    priority // Above-the-fold images
    placeholder="blur"
    blurDataURL="data:image/jpeg;base64,..."
  />
)
```

### Code Splitting Strategy
- Route-based splitting (automatic with App Router)
- Component-based splitting for heavy components
- Dynamic imports for non-critical features
```typescript
const HeavyComponent = dynamic(() => import('./HeavyComponent'), {
  loading: () => <ComponentSkeleton />,
  ssr: false // If not needed for SEO
})
```

### Bundle Optimization
- Tree-shaking enabled for all dependencies
- Minimize bundle size with webpack-bundle-analyzer
- Use ES modules for better tree-shaking
- Implement service worker for caching strategy

## 💻 Code Quality Best Practices

### TypeScript Standards
```typescript
// Strict type definitions
interface ServiceCard {
  readonly id: string;
  title: string;
  description: string;
  technologies: readonly Technology[];
  performance: {
    priority: 'high' | 'medium' | 'low';
    renderOrder: number;
  };
}

// Use branded types for better type safety
type EmailAddress = string & { readonly brand: unique symbol };
```

### Component Architecture
```typescript
// Compound component pattern example
const Navigation = {
  Root: NavigationRoot,
  Item: NavigationItem,
  Menu: NavigationMenu
} as const;

// Usage with proper composition
<Navigation.Root>
  <Navigation.Menu>
    <Navigation.Item href="#services">Services</Navigation.Item>
  </Navigation.Menu>
</Navigation.Root>
```

### Error Boundaries and Resilience
- Implement error boundaries for each major section
- Graceful degradation for JavaScript failures
- Proper loading states and error messages
- Progressive enhancement approach

### Testing Strategy
- Unit tests for utility functions
- Integration tests for critical user flows
- Visual regression testing for UI components
- Performance testing with Lighthouse CI

## 🎨 UX/UI Best Practices

### Accessibility (WCAG 2.1 AA Compliance)
```typescript
// Semantic HTML and ARIA labels
<button
  aria-label="Toggle navigation menu"
  aria-expanded={isOpen}
  aria-controls="navigation-menu"
  className={styles.menuToggle}
>
  <span className="sr-only">Menu</span>
  <HamburgerIcon />
</button>
```

### Interaction Design
- **Touch Targets**: Minimum 44px × 44px for mobile
- **Focus Management**: Visible focus indicators, logical tab order
- **Keyboard Navigation**: Full keyboard accessibility
- **Gesture Support**: Swipe navigation where appropriate

### Motion and Animation Guidelines
```css
/* Respect user preferences */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}

/* Smooth, purposeful animations */
.card {
  transition: transform 0.2s ease-out, box-shadow 0.2s ease-out;
}

.card:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
}
```

### Responsive Design Principles
- Mobile-first CSS architecture
- Fluid typography using `clamp()`
- Container queries for component-level responsiveness
- Optimized images for different screen densities

### Loading and Feedback States
```typescript
// Progressive loading with skeleton screens
const SkeletonCard = () => (
  <div className={styles.skeleton} aria-label="Loading content">
    <div className={styles.skeletonTitle} />
    <div className={styles.skeletonText} />
  </div>
);

// Optimistic UI updates
const ContactForm = () => {
  const [isSubmitting, setIsSubmitting] = useState(false);
  // Implementation with immediate feedback
};
```

## Key Components & Sections

### Navigation System
- Sticky header with smooth scroll and offset calculation
- Breadcrumb navigation for deep sections
- Active section highlighting with intersection observer
- Mobile-optimized hamburger menu with proper animations

### Hero Section
- Optimized SVG animations with reduced complexity
- Terminal simulation with realistic typing effects
- Progressive image loading with blur-up technique
- Call-to-action with clear value proposition

### Services Grid
- 6-card responsive grid with CSS Grid
- Hover states with hardware acceleration
- Progressive disclosure of information
- Accessible card interactions

### Technology Showcase
- Categorized display with filtering capabilities
- Icon optimization and lazy loading
- Semantic grouping with proper headings
- Progressive enhancement for interactions

## Performance Monitoring

### Core Metrics Tracking
```javascript
// Web Vitals monitoring
import { getCLS, getFID, getFCP, getLCP, getTTFB } from 'web-vitals';

function sendToAnalytics(metric) {
  // Send metrics to your analytics service
  analytics.track('Web Vitals', {
    name: metric.name,
    value: metric.value,
    id: metric.id,
  });
}

getCLS(sendToAnalytics);
getFID(sendToAnalytics);
getFCP(sendToAnalytics);
getLCP(sendToAnalytics);
getTTFB(sendToAnalytics);
```

### Performance Budget
- JavaScript bundle: < 150KB gzipped
- CSS bundle: < 50KB gzipped
- Images: WebP format, optimized sizes
- Fonts: Preload critical fonts, font-display: swap

## Configuration & Optimization

### Next.js Configuration
```javascript
// next.config.js optimizations
const nextConfig = {
  experimental: {
    optimizeCss: true,
    optimizePackageImports: ['react-icons']
  },
  images: {
    formats: ['image/avif', 'image/webp'],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384]
  },
  compress: true,
  poweredByHeader: false,
  generateEtags: false
};
```

### CSS Optimization
- Critical CSS inlining for above-the-fold content
- CSS containment for performance isolation
- Custom property optimization
- Purge unused CSS in production

## Development Workflow

### Code Quality Gates
1. **Pre-commit**: Lint, type-check, format
2. **Pre-push**: Run tests, build validation
3. **CI/CD**: Performance testing, accessibility audit
4. **Deployment**: Lighthouse CI, bundle analysis

### Development Guidelines
- Follow conventional commits for clear history
- Use feature flags for experimental features  
- Implement proper logging and monitoring
- Document component APIs with JSDoc
- Regular dependency updates with security scanning

## Accessibility Checklist

- [ ] Semantic HTML structure
- [ ] ARIA labels and roles
- [ ] Keyboard navigation support
- [ ] Screen reader compatibility
- [ ] Color contrast ratios (4.5:1 minimum)
- [ ] Focus management
- [ ] Alternative text for images
- [ ] Reduced motion preferences
- [ ] Proper heading hierarchy
- [ ] Form validation and error messages

## Browser Support

- **Modern Browsers**: Chrome 90+, Firefox 88+, Safari 14+, Edge 90+
- **Progressive Enhancement**: Core functionality works without JavaScript
- **Polyfills**: Minimal polyfills for critical features only
- **Testing**: Cross-browser testing with BrowserStack

This enhanced documentation ensures the portfolio maintains high standards for performance, accessibility, and user experience while providing clear guidelines for future development.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tecnomage) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
