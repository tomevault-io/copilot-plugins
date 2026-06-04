## locales

> Locales coding standards and best practices guide

# Translation Development Standards

Auto-attached when working in `locales/` directory.

## File Structure

```
locales/
├── en.default.json     # English (required)
├── es.json             # Spanish
├── fr.json             # French
└── de.json             # German
```

## Key Organization

**Hierarchical Structure:**
```json
{
  "general": {
    "meta": {
      "title": "{{ shop_name }}",
      "description": "{{ shop_description }}"
    },
    "accessibility": {
      "skip_to_content": "Skip to content",
      "close": "Close"
    }
  },
  "products": {
    "add_to_cart": "Add to cart",
    "quick_view": "Quick view",
    "price": {
      "regular": "Regular price",
      "sale": "Sale price",
      "unit": "Unit price"
    }
  }
}
```
**Usage**
```liquid
{{ 'general.meta.title' | t: shop_name: shop.name }}
{{ 'general.meta.description' | t: shop_description: shop.description }}
```

## Translation Guidelines

**Key Naming:**
- Use descriptive, hierarchical keys
- Maximum 3 levels deep
- Use snake_case for key names
- Group related translations

**Content Rules:**
- Keep text concise for UI elements
- Use variables for dynamic content
- Consider character limits
- Maintain consistent terminology

**Schema Translations:**

Use the rules outlined in our [typescript schema](schemas/schema.d.ts) to determine the appropriate key structure for schema translations.

---
> Source: [EcomExperts-io/Base](https://github.com/EcomExperts-io/Base) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
