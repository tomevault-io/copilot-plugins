## cr-engine

> - Prioritize Largest Contentful Paint (LCP) optimization

# Performance Optimization in Astro

## Core Web Vitals
- Prioritize Largest Contentful Paint (LCP) optimization
- Minimize First Input Delay (FID) and Cumulative Layout Shift (CLS)
- Use Lighthouse and WebPageTest for performance auditing
- Implement performance budgets and monitoring

## Static Generation
- Leverage Astro's static generation capabilities
- Use `getStaticPaths()` for pre-rendering dynamic routes
- Minimize client-side JavaScript with partial hydration
- Generate static assets at build time when possible

## Image Optimization
- Use Astro's built-in Image component for optimized delivery
- Implement proper lazy loading for images
- Use appropriate image formats (WebP, AVIF) with fallbacks
- Optimize image dimensions and compression

## JavaScript Optimization
- Use `client:*` directives judiciously:
  - `client:load` - for immediately needed interactivity
  - `client:idle` - for non-critical interactivity
  - `client:visible` - for components that should hydrate when visible
  - `client:media` - for responsive hydration
  - `client:only` - for framework-specific components

## CSS Optimization
- Use Tailwind's purge process to remove unused styles
- Minimize CSS bundle size with proper utility usage
- Implement critical CSS inlining for above-the-fold content
- Use CSS custom properties for dynamic values

## Asset Optimization
- Implement proper caching strategies for static assets
- Use CDN for global asset delivery
- Optimize font loading with font-display strategies
- Minimize HTTP requests with asset bundling

## Data Fetching
- Use `Astro.glob()` for efficient local file processing
- Implement proper caching for external API calls
- Use static data generation at build time
- Minimize runtime data fetching

## Bundle Optimization
- Monitor bundle sizes with build analysis tools
- Implement code splitting for large components
- Use tree shaking to eliminate dead code
- Optimize third-party library usage

## Caching Strategies
- Implement proper cache headers for static assets
- Use service workers for offline functionality
- Implement browser caching for frequently accessed resources
- Use CDN caching for global performance

## Monitoring and Metrics
- Implement Real User Monitoring (RUM) for performance tracking
- Use Core Web Vitals reporting
- Monitor bundle sizes and loading times
- Track user experience metrics

## Build Optimization
- Use production builds with proper optimization flags
- Implement incremental builds for faster development
- Use parallel processing for build tasks
- Optimize build pipeline with proper tooling

## SEO Performance
- Implement proper meta tags and structured data
- Use sitemap generation for better crawling
- Optimize for Core Web Vitals in search rankings
- Implement proper canonical URLs and redirects

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nirus) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
