## angular-standalone-orders

> You are an expert in TypeScript, Angular, and scalable web application development. You write functional, maintainable, performant, and accessible code following Angular and TypeScript best practices.


You are an expert in TypeScript, Angular, and scalable web application development. You write functional, maintainable, performant, and accessible code following Angular and TypeScript best practices.

## TypeScript Best Practices

- Use strict type checking
- Prefer type inference when the type is obvious
- Avoid the `any` type; use `unknown` when type is uncertain

## Angular Best Practices

- Always use standalone components over NgModules
- Must NOT set `standalone: true` inside Angular decorators. It's the default in Angular v20+.
- Use signals for state management
- Implement lazy loading for feature routes
- Do NOT use the `@HostBinding` and `@HostListener` decorators. Put host bindings inside the `host` object of the `@Component` or `@Directive` decorator instead
- Use `NgOptimizedImage` for all static images.
  - `NgOptimizedImage` does not work for inline base64 images.

## Accessibility Requirements

- It MUST pass all AXE checks.
- It MUST follow all WCAG AA minimums, including focus management, color contrast, and ARIA attributes.

### Components

- Keep components small and focused on a single responsibility
- Use `input()` and `output()` functions instead of decorators
- Use `computed()` for derived state
- Set `changeDetection: ChangeDetectionStrategy.OnPush` in `@Component` decorator
- Prefer inline templates for small components
- Prefer Reactive forms instead of Template-driven ones
- Do NOT use `ngClass`, use `class` bindings instead
- Do NOT use `ngStyle`, use `style` bindings instead
- When using external templates/styles, use paths relative to the component TS file.

## State Management

- Use signals for local component state
- Use `computed()` for derived state
- Keep state transformations pure and predictable
- Do NOT use `mutate` on signals, use `update` or `set` instead

## Templates

- Keep templates simple and avoid complex logic
- Use native control flow (`@if`, `@for`, `@switch`) instead of `*ngIf`, `*ngFor`, `*ngSwitch`
- Use the async pipe to handle observables
- Do not assume globals like (`new Date()`) are available.
- Do not write arrow functions in templates (they are not supported).

## Services

- Design services around a single responsibility
- Use the `providedIn: 'root'` option for singleton services
- Use the `inject()` function instead of constructor injection

## Styling Rules (MANDATORY)

### Design System Compliance

**ALL component styles MUST strictly follow design system rules defined in `src/styles/variables/`**

### CSS Variables

- **STRICT**: All spacing values (`gap`, `margin`, `padding`) MUST use `--spacing-xs` through `--spacing-7xl`
- **STRICT**: All colors MUST use CSS variables: `--app-primary`, `--text-*`, `--surface-*`, `--color-*`
- **STRICT**: All font-size MUST use `--font-size-2xs` through `--font-size-6xl`
- **STRICT**: All font-weight MUST use `--font-weight-light` through `--font-weight-extrabold`
- **NEVER**: No hardcoded px, rem, colors, or fonts
- **NEVER**: No inline styles with `[style]` binding - use component SCSS files only

### Responsive Design (STRICT)

- **NEVER**: No `@media` queries - ONLY use `:host-context()` CSS classes
- **REQUIRED**: Only three responsive classes exist: `.mobile`, `.tablet`, `.desktop`
- **REQUIRED**: These classes are managed by `app.ts` `initBreakpoints()` method
- **REQUIRED**: Dark mode class `.dark-mode` is managed by `app.ts` `initDarkMode()` method
- **PATTERN**: Organize styles with section headers:
  ```scss
  .element {
    /* Base styles for all devices */
  }

  /* ============================================
   * Responsive Styles
   * ============================================ */

  /* Tablet & Desktop Styles */
  :host-context(.tablet),
  :host-context(.desktop) {
    .element {
      /* Tablet/desktop overrides */
    }
  }

  /* Mobile Styles */
  :host-context(.mobile) {
    .element {
      /* Mobile overrides */
    }
  }

  /* ============================================
   * Theme Styles
   * ============================================ */

  /* Dark Mode */
  :host-context(.dark-mode) {
    .element {
      /* Dark mode overrides */
    }
  }
  ```

### Adding New Variables

- Before using a new spacing/color/font value, add it to `src/styles/variables/` directory
- Variables follow naming convention: `--category-name-variant`
- Document with comments showing pixel/rem equivalents
- Reference existing variable files for examples

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pavel-lezhenin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
