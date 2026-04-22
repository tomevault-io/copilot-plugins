## web

> 1. **Always use $ms namespace** - All framework functionality must be accessed through `$ms`


# MS Query Framework Rules

## Core Principles

1. **Always use $ms namespace** - All framework functionality must be accessed through `$ms`
2. **Prefer UI components over manual DOM creation** - Use `$ms.ui.create()` for consistent styling
3. **Follow Bootstrap conventions** - All UI must use Bootstrap-like classes and patterns
4. **No external dependencies** - Framework must be self-contained

## Required Usage Patterns

### When working with DOM:
- Use `$()` selector for DOM manipulation
- Use `$ms.query` as alias for `$`
- Chain methods like jQuery: `$('#app').addClass('active').show()`

### When creating UI elements:
- ALWAYS use `$ms.ui.create()` for components
- Use Bootstrap-like CSS classes from `ms.css`
- Follow component patterns: button, input, card, alert, badge, progress, spinner

### When styling:
- Use CSS variables from `ms.css` (e.g., `var(--ms-primary)`)
- Apply Bootstrap utility classes (e.g., `ms-btn-primary`, `ms-form-control`)
- Don't write custom CSS unless absolutely necessary

## Component Creation Rules

### Buttons:
```javascript
// CORRECT:
var btn = $ms.ui.create('button', {
    text: 'Click Me',
    type: 'primary',  // primary, secondary, success, danger, warning, info, light, dark
    size: 'lg',      // sm, lg
    outline: false,   // true for outline style
    block: false,    // true for full width
    click: handler
});

// WRONG:
var btn = document.createElement('button');
btn.innerHTML = 'Click Me';
btn.style.backgroundColor = '#007bff';
```

### Forms:
```javascript
// CORRECT:
var input = $ms.ui.create('input', {
    label: 'Email',
    type: 'email',
    placeholder: 'Enter email',
    change: handler
});

// WRONG:
var input = document.createElement('input');
input.type = 'email';
input.placeholder = 'Enter email';
```

### Cards:
```javascript
// CORRECT:
var card = $ms.ui.create('card', {
    title: 'Card Title',
    content: 'Card content',
    header: 'Optional header',
    footer: 'Optional footer'
});

// WRONG:
var card = document.createElement('div');
card.className = 'card';
card.innerHTML = '<div class="card-body">...</div>';
```

## CSS Usage Rules

### Grid System:
- Use `ms-container` for responsive containers
- Use `ms-row` and `ms-col-*` for grid layout
- Follow 12-column grid system

### Component Classes:
- Buttons: `ms-btn ms-btn-{type} ms-btn-{size}`
- Forms: `ms-form-group`, `ms-form-label`, `ms-form-control`
- Alerts: `ms-alert ms-alert-{type}`
- Cards: `ms-card`, `ms-card-body`, `ms-card-header`, `ms-card-footer`
- Badges: `ms-badge ms-badge-{type}`

### Utility Classes:
- Spacing: `ms-m-*`, `ms-p-*`, `ms-mt-*`, `ms-mb-*`
- Display: `ms-d-none`, `ms-d-block`, `ms-d-flex`
- Text: `ms-text-center`, `ms-text-left`, `ms-text-right`

## Prohibited Patterns

1. **Never create UI elements with raw DOM APIs** - Always use `$ms.ui.create()`
2. **Never write inline styles** - Use CSS classes and variables
3. **Never use external libraries** - Framework must be self-contained
4. **Never bypass $ms namespace** - All functionality through `$ms`

## Required Bootstrap Compliance

All UI must:
- Use Bootstrap color scheme (primary, secondary, success, danger, warning, info, light, dark)
- Follow Bootstrap spacing and sizing conventions
- Implement responsive design patterns
- Use Bootstrap utility classes for common tasks

## Module System Rules

When creating modules:
```javascript
// Register modules through registry
$ms.moduleRegistry.register('myModule', {
    name: 'My Module',
    version: '1.0.0',
    init: function() {
        // Module initialization
    },
    destroy: function() {
        // Cleanup
    }
});
```

## Error Handling

Always use `$ms.core.log()` for logging:
```javascript
$ms.core.log('Error message', 'error');
$ms.core.log('Warning message', 'warn');
$ms.core.log('Info message', 'info');
```

## File Organization

- Framework code in `js/ms.js`
- Styles in `css/ms.css`
- Application logic in `js/app.js`
- Documentation in `README.md`
- Examples in `index.html`

## Performance Guidelines

1. Use component caching when appropriate
2. Minimize DOM manipulation
3. Use event delegation for multiple elements
4. Lazy load non-critical components

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sternonline) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
