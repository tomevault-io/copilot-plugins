## agents

> - Use APCA (Advanced Perceptual Contrast Algorithm) for accurate perceptual contrast measurements

# CLAUDE.md - Development Guidelines for AI Assistants

## UI/UX Design Guidelines (Vercel Design Principles)

### Color & Contrast
- Use APCA (Advanced Perceptual Contrast Algorithm) for accurate perceptual contrast measurements
- Increase contrast on `:hover`, `:active`, and `:focus` states for better interactivity feedback
- Ensure sufficient color contrast for accessibility (WCAG AA/AAA compliance)
- Never rely solely on color to convey information or status - always provide additional indicators

### Typography
- Use tabular numbers (`font-variant-numeric: tabular-nums`) for data comparisons and tables
- Prefer typographic (curly) quotes ("" '') over straight quotes in content
- Avoid widows and orphans in text blocks using CSS properties or manual adjustments
- Maintain consistent font hierarchy and sizing throughout the application

### Spacing & Layout
- Practice "optical alignment" - adjust elements by ±1px for perceptual balance rather than mathematical precision
- Ensure deliberate alignment to grid, baseline, or edges for visual consistency
- Balance contrast between text and icons for optimal readability
- Design for responsive coverage (mobile, laptop, ultra-wide) with breakpoints at 640px, 768px, 1024px, 1280px
- Minimize layout shifts during loading and interactions (optimize CLS scores)

### Interaction Design
- **Keyboard accessibility is paramount** - all interactive elements must be keyboard accessible
- Provide clear, visible focus states (never use `outline: none` without replacement)
- Create "forgiving interactions" with generous hit targets (minimum 44x44px on mobile, 24x24px on desktop)
- Implement predictable interaction patterns following platform conventions
- Use optimistic UI updates where appropriate to improve perceived performance
- Deep-link application states for better navigation and shareability

### Accessibility Standards
- Follow WAI-ARIA Authoring Patterns for all custom components
- Use semantic HTML elements (`nav`, `main`, `article`, `section`, `header`, `footer`, etc.)
- Provide text alternatives for all icons and images using `alt` attributes or `aria-label`
- Ensure all forms have proper labels, placeholders, and error messages
- Test with screen readers (NVDA, JAWS, VoiceOver) and keyboard navigation

### Performance & UX
- Optimize for minimal layout work and repaints using CSS containment and will-change
- Implement loading skeletons instead of spinners where possible for better perceived performance
- Persist scroll positions across navigation using scroll restoration
- Handle all user states: empty, loading, error, success, dense data
- Provide immediate feedback for user actions through micro-interactions

## Code Style Guidelines

### React/Next.js Components
- Use functional components with TypeScript
- Implement proper error boundaries
- Use React.memo() for expensive components
- Prefer composition over prop drilling
- Keep components focused and single-purpose

### CSS/Styling
- Use CSS Modules or CSS-in-JS for component styling
- Implement design tokens for consistency
- Use CSS custom properties for theming
- Prefer logical properties (inline-start vs left)
- Implement responsive design using modern CSS (Grid, Flexbox, Container Queries)

### State Management
- Use appropriate state management based on scope (useState, useReducer, Context, Zustand)
- Implement optimistic updates for better UX
- Handle loading and error states consistently
- Persist critical state to localStorage when appropriate

### Testing
- Write unit tests for utility functions
- Implement integration tests for critical user flows
- Test accessibility with automated tools (axe-core)
- Perform visual regression testing for UI components

## Development Workflow

### Before Starting
1. Review existing components and patterns
2. Check accessibility requirements
3. Plan responsive behavior
4. Consider performance implications

### During Development
1. Use semantic HTML
2. Implement keyboard navigation
3. Add ARIA attributes where needed
4. Test with screen readers
5. Validate color contrast

### Before Completion
1. Run accessibility audit (Lighthouse, axe)
2. Test keyboard navigation
3. Verify responsive behavior
4. Check performance metrics
5. Update documentation

## Commands to Run
```bash
# Linting and type checking
npm run lint
npm run typecheck

# Testing
npm run test
npm run test:a11y

# Build verification
npm run build
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jonymusky)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jonymusky)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
