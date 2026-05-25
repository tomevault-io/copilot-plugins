## 2026-performance-best-practices

> 2026 web performance optimization standards


# 2026 Performance Best Practices

## Images (Critical)

```typescript
// ✅ GOOD: Next.js Image with AVIF/WebP
import Image from 'next/image';

<Image
  src="/property.jpg"
  alt="Beautiful home"
  width={800}
  height={600}
  loading="lazy"
  sizes="(max-width: 768px) 100vw, 800px"
/>

// ❌ BAD: Plain img tag
<img src="/property.jpg" />
```

**Config (next.config.js):**
```javascript
images: {
  formats: ['image/avif', 'image/webp'],
  minimumCacheTTL: 31536000, // 1 year
}
```

## Caching Strategy

```javascript
// Page revalidation
export const revalidate = 3600; // 1 hour ISR

// Static generation
export async function generateStaticParams() {
  // Pre-render critical paths
}
```

## Core Web Vitals Targets (2026)

- **LCP** (Largest Contentful Paint): <2.5s
- **FID** (First Input Delay): <100ms
- **CLS** (Cumulative Layout Shift): <0.1
- **FCP** (First Contentful Paint): <1.8s
- **TBT** (Total Blocking Time): <300ms

## Accessibility (WCAG 2.1)

```typescript
// ✅ Respect motion preferences
import { useReducedMotion } from '@/hooks/useReducedMotion';

const prefersReducedMotion = useReducedMotion();

<div className={prefersReducedMotion ? '' : 'animate-fade-in'}>

// ✅ Keyboard navigation
<button
  onClick={handleClick}
  onKeyDown={(e) => {
    if (e.key === 'Enter' || e.key === ' ') {
      e.preventDefault();
      handleClick();
    }
  }}
  aria-label="Descriptive label"
>

// ✅ ARIA attributes
<nav role="navigation" aria-label="Main navigation">
```

## Code Splitting

```typescript
// ✅ Dynamic imports for heavy components
import dynamic from 'next/dynamic';

const Map = dynamic(() => import('@/components/Map'), {
  loading: () => <MapSkeleton />,
  ssr: false,
});
```

## Security Headers (Required)

```javascript
// next.config.js headers()
{
  key: 'Strict-Transport-Security',
  value: 'max-age=63072000; includeSubDomains; preload'
},
{
  key: 'Content-Security-Policy',
  value: "default-src 'self'; ..."
}
```

---
> Source: [LetMeHelpYouREALTY/opportunityzonespecialists.com](https://github.com/LetMeHelpYouREALTY/opportunityzonespecialists.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
