## korskavireis

> All design code should primarily work on mobile first. This means:

# Mobile-First Design Guidelines

All design code should primarily work on mobile first. This means:

1. **CSS Media Queries**: Start with mobile styles as the base, then use `@media (min-width: 768px)` for desktop enhancements
2. **Touch-Friendly**: All interactive elements should be at least 44px tall for easy touch interaction
3. **Horizontal Scrolling**: Use horizontal scroll containers for content that doesn't fit vertically
4. **Responsive Images**: Images should scale properly on all screen sizes
5. **Typography**: Use readable font sizes (minimum 16px) and proper line heights
6. **Spacing**: Use consistent spacing that works well on small screens
7. **Performance**: Optimize for mobile performance with minimal JavaScript and efficient CSS

## Key CSS Patterns:
- Use `flexbox` for layout
- Implement `overflow-x: auto` for horizontal scrolling
- Use `scroll-snap-type: x mandatory` for smooth scrolling
- Ensure `touch-action: pan-x` for proper touch scrolling
- Test on actual mobile devices when possible
description:
globs:
alwaysApply: true
---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/anarkistix) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
