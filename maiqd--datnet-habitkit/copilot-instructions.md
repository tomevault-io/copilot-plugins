## datnet-habitkit

> Mobile Development Best Practices


# Mobile Development Best Practices

## Core Principles
- **Mobile-First**: Design for mobile devices first, then enhance for larger screens
- **Touch-Friendly**: Ensure all interactive elements meet minimum touch target sizes (44px)
- **Performance**: Optimize for mobile hardware constraints and slower networks
- **Accessibility**: Follow WCAG guidelines for mobile accessibility

## Viewport and Meta Tags
Always include proper viewport configuration:
```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover" />
```

## Touch Interaction Guidelines

### Touch Target Sizes
- Minimum touch target: 44px × 44px
- Recommended touch target: 48px × 48px
- Spacing between touch targets: 8px minimum

### Touch Event Handling
```javascript
// Prevent double-tap zoom
var lastTouchEnd = 0;
document.addEventListener('touchend', function(event) {
    var now = (new Date()).getTime();
    if (now - lastTouchEnd <= 300) {
        event.preventDefault();
    }
    lastTouchEnd = now;
}, false);
```

## Performance Optimization

### CSS Performance
- Use hardware-accelerated properties (`transform`, `opacity`)
- Avoid expensive effects (`backdrop-filter`, complex `box-shadow`)
- Implement CSS containment for isolated components
- Use `will-change` sparingly and remove when not needed

### JavaScript Performance
- Minimize DOM manipulation
- Use `requestAnimationFrame` for animations
- Implement proper cleanup for event listeners
- Use efficient selectors and avoid deep nesting

### Image Optimization
- Use appropriate image formats (WebP, AVIF when supported)
- Implement responsive images with `srcset`
- Lazy load images below the fold
- Optimize image sizes for different screen densities

## Responsive Design Patterns

### Breakpoint Strategy
```css
/* Mobile first approach */
/* Base styles for mobile (320px+) */
.component { }

/* Small mobile (375px+) */
@media (min-width: 375px) { }

/* Tablet (768px+) */
@media (min-width: 768px) { }

/* Desktop (1024px+) */
@media (min-width: 1024px) { }

/* Large desktop (1440px+) */
@media (min-width: 1440px) { }
```

### Fluid Typography
```css
.responsive-text {
    font-size: clamp(0.875rem, 2.5vw, 1.125rem);
    line-height: clamp(1.4, 2vw, 1.6);
}
```

## Platform-Specific Considerations

### iOS Safari
- Disable expensive CSS effects (`backdrop-filter`, complex transforms)
- Use `-webkit-overflow-scrolling: touch` for smooth scrolling
- Handle iOS-specific CSS custom property issues
- Implement proper touch event handling

### Android Chrome
- Test on various Android versions and screen sizes
- Handle different input methods (touch, stylus, keyboard)
- Consider Android-specific UI patterns

## Accessibility Guidelines

### Touch Accessibility
- Ensure sufficient color contrast (4.5:1 minimum)
- Provide alternative input methods for touch gestures
- Use proper ARIA labels for interactive elements
- Implement keyboard navigation support

### Screen Reader Support
- Use semantic HTML elements
- Provide descriptive alt text for images
- Implement proper heading hierarchy
- Use ARIA landmarks for navigation

## Testing Strategy

### Device Testing
- Test on actual devices, not just simulators
- Test on various screen sizes and orientations
- Test with different input methods
- Test with accessibility features enabled

### Performance Testing
- Use Chrome DevTools Performance tab
- Test on slower networks (3G simulation)
- Monitor Core Web Vitals
- Test battery impact on mobile devices

## Common Mobile Issues

### Zoom Prevention
- Use proper viewport settings
- Implement touch event handling
- Avoid small touch targets

### Scrolling Issues
- Enable hardware-accelerated scrolling
- Handle scroll momentum
- Implement proper scroll boundaries

### Performance Problems
- Minimize JavaScript execution time
- Optimize CSS animations
- Implement proper image loading strategies
- Use efficient data structures

## Development Workflow

### Mobile-First Development
1. Start with mobile layout and functionality
2. Test on actual mobile devices
3. Enhance for larger screens progressively
4. Validate performance on mobile networks

### Progressive Enhancement
- Core functionality works without JavaScript
- Enhanced features require JavaScript
- Graceful degradation for unsupported features
- Fallbacks for modern CSS features

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/maiqd) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
