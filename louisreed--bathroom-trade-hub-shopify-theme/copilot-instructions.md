## bathroom-trade-hub-shopify-theme

> - **Project**: Bathroom Trade Hub Shopify Theme

# Bathroom Trade Hub - Shopify Theme Development Rules

## Project Context

- **Project**: Bathroom Trade Hub Shopify Theme
- **Business**: Dorset-based bathroom supplies for trade professionals and DIY enthusiasts
- **Live Site**: bathroomtradehub.co.uk
- **Theme Version**: 4.3.0+ (Latest Shopify standards)
- **Target Audience**: Trade professionals, contractors, DIY enthusiasts
- **Geographic Focus**: UK market with local Dorset operations

## Technical Stack

- **Templating**: Shopify Liquid
- **JavaScript**: Vanilla ES6+ (no jQuery)
- **CSS**: Tailwind-inspired utility classes + custom CSS
- **Languages**: Multi-language support (EN, DE, ES, FR, IT, VI)
- **Performance**: Lazy loading, CDN optimization, minified assets
- **Architecture**: Component-based sections and snippets

## File Structure Rules

### Directory Organization

```
/assets/          # CSS, JS, images, fonts
/config/          # Theme settings (settings_data.json, settings_schema.json)
/layout/          # Base templates (theme.liquid, password.liquid)
/locales/         # Translation files (.json, .schema.json)
/sections/        # Reusable page sections (.liquid)
/snippets/        # Reusable components (.liquid)
/templates/       # Page templates (.json, .liquid)
/blocks/          # Custom blocks (if any)
```

### File Naming Conventions

- Use kebab-case for all files: `footer-contact.liquid`
- Prefix snippets with purpose: `product-card.liquid`, `footer-logo.liquid`
- Use descriptive names: `main-product.liquid` not `product.liquid`
- Translation files: `en.default.json`, `de.schema.json`

## Shopify Development Guidelines

### Liquid Best Practices

- Always use `{%- liquid -%}` syntax for multi-line logic
- Use `| escape` for user-generated content
- Implement proper `{% schema %}` for all sections
- Use `{% comment %}` for complex logic documentation
- Prefer `assign` over `capture` for simple variables
- Use `render` instead of deprecated `include`

### Schema Structure

```liquid
{% schema %}
{
  "name": "t:sections.section_name.name",
  "settings": [
    {
      "type": "header",
      "content": "t:sections.global.settings.header__content.content"
    },
    {
      "type": "range",
      "id": "setting_id",
      "label": "t:sections.section_name.settings.setting_id.label",
      "default": 16,
      "min": 12,
      "max": 24,
      "step": 1,
      "unit": "px"
    }
  ]
}
{% endschema %}
```

### Performance Optimization

- Use `loading="lazy"` for images below the fold
- Implement `preload` for critical resources
- Use `defer` for non-critical JavaScript
- Minimize liquid loops and complex calculations
- Use `{% liquid %}` blocks for better performance
- Implement proper image sizing with `image_url` filters

### Responsive Design

- Mobile-first approach (320px+)
- Breakpoints: sm (640px), md (768px), lg (1024px), xl (1280px)
- Use CSS Grid and Flexbox for layouts
- Test on actual devices, not just browser tools
- Optimize touch targets (44px minimum)

## Business Logic Rules

### Trade Account Features

- Implement wholesale pricing display logic
- Support bulk ordering interfaces
- Handle customer group-based pricing
- Show trade-specific product information
- Implement account verification workflows

### Product Handling

- Support high-variant products (bathroom fixtures)
- Handle complex product specifications
- Implement product bundles and volume pricing
- Show real-time inventory levels
- Support product recommendations

### UK Market Specifics

- Handle UK currency (GBP) as primary
- Support UK address formats
- Implement UK VAT calculations
- Handle UK shipping zones
- Support local pickup for Dorset area

## Coding Standards

### HTML

- Use semantic HTML5 elements
- Implement proper heading hierarchy (h1 → h6)
- Use ARIA labels for accessibility
- Include proper meta tags for SEO
- Use microdata/schema markup for products

### CSS

- Use BEM methodology where applicable
- Implement CSS custom properties for theming
- Use mobile-first media queries
- Avoid `!important` declarations
- Group related styles together
- Use consistent naming conventions

### JavaScript

- Use modern ES6+ syntax
- Implement proper error handling
- Use event delegation for dynamic content
- Avoid inline event handlers
- Use async/await for promises
- Implement proper module patterns

## Accessibility (WCAG 2.1 AA)

- Maintain proper color contrast ratios
- Use semantic markup
- Implement keyboard navigation
- Add screen reader support
- Use proper ARIA attributes
- Test with screen readers
- Ensure focus indicators are visible

## Multi-Language Support

- Use translation keys for all text: `{{ 'key' | t }}`
- Implement proper RTL support where needed
- Use locale-specific formatting for dates/numbers
- Test all languages thoroughly
- Keep translation files organized by feature
- Use descriptive translation keys

## Security Best Practices

- Sanitize all user inputs
- Use HTTPS for all resources
- Implement proper CSRF protection
- Validate form data server-side
- Use secure cookie settings
- Implement rate limiting for forms
- Regular security audits

## Performance Targets

- First Contentful Paint: < 2.5s
- Largest Contentful Paint: < 4s
- Cumulative Layout Shift: < 0.1
- Time to Interactive: < 5s
- Mobile Performance Score: > 90
- Desktop Performance Score: > 95

## Testing Guidelines

- Test on real devices (iPhone, Android, iPad)
- Use different network conditions
- Test with screen readers
- Validate HTML markup
- Test in all supported browsers
- Check performance with Lighthouse
- Test multi-language functionality

## Common Patterns

### Section Structure

```liquid
{%- doc -%}
  Section description and usage
{%- enddoc -%}

<style>
  #shopify-section-{{ section.id }} {
    --section-padding-top: {{ section.settings.padding_top }}px;
    --section-padding-bottom: {{ section.settings.padding_bottom }}px;
  }
</style>

<div class="section section--padding" id="section-{{ section.id }}">
  <div class="page-width">
    <!-- Section content -->
  </div>
</div>
```

### Snippet Structure

```liquid
{%- doc -%}
  Snippet description and parameters
  @param {object} param_name - Description
{%- enddoc -%}

{%- liquid
  assign param_default = param_name | default: 'default_value'
-%}

<div class="component-name">
  <!-- Component content -->
</div>
```

### Error Handling

```liquid
{%- liquid
  assign product = collections.all.products[handle]
  if product == blank
    assign product = collections.all.products.first
  endif
-%}
```

## Anti-Patterns to Avoid

- Don't use inline styles
- Avoid complex nested loops in liquid
- Don't use jQuery (use vanilla JavaScript)
- Avoid hard-coded text (use translations)
- Don't use `!important` in CSS
- Avoid blocking JavaScript in `<head>`
- Don't ignore accessibility requirements
- Avoid non-semantic HTML

## Debug and Development

### Liquid Debugging

```liquid
<!-- Debug output -->
{{ variable | json }}
{% comment %} Debug comment {% endcomment %}
```

### Console Logging

```javascript
// Development only
if (Shopify.designMode) {
  console.log("Debug info:", data);
}
```

### Performance Monitoring

- Use Shopify's built-in performance tools
- Monitor Core Web Vitals
- Check liquid render times
- Monitor third-party script impact

## Third-Party Integrations

- Minimize external scripts
- Use async loading for non-critical scripts
- Implement proper error handling for external APIs
- Test integration failures gracefully
- Document all third-party dependencies

## Deployment Guidelines

- Test in development environment first
- Backup theme before major changes
- Use version control for all changes
- Deploy during low-traffic periods
- Monitor for errors after deployment
- Have rollback plan ready

## Maintenance Practices

- Regular performance audits
- Update dependencies regularly
- Monitor for liquid deprecations
- Keep documentation updated
- Regular accessibility testing
- Monitor error logs
- Update translations as needed

## Business-Specific Features

- Trade account verification flows
- Bulk order processing
- Professional product specifications
- Local delivery calculations
- UK-specific compliance requirements
- Trade pricing displays
- Product bundle configurations

## Code Quality Checks

- Validate all HTML
- Check CSS for unused rules
- Lint JavaScript code
- Test liquid syntax
- Verify translation completeness
- Check image optimization
- Validate structured data

## Emergency Procedures

- Keep backup of working theme
- Document rollback procedures
- Have emergency contact information
- Know how to disable problematic features
- Monitor site uptime and performance
- Have debugging tools readily available

Remember: This is a business-critical e-commerce site serving trade professionals. Prioritize reliability, performance, and user experience over experimental features.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Louisreed) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
