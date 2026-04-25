## collabiome-shopify

> Development standards and best practices for creating, configuring, and styling theme blocks, including static and nested blocks, schema configuration, CSS, and usage examples

# Theme blocks development standards

Follow [Shopify's theme blocks documentation](mdc:https:/shopify.dev/docs/storefronts/themes/architecture/blocks/theme-blocks/quick-start?framework=liquid).

## Theme block fundamentals

Theme blocks are reusable components defined at the theme level that can be:
- Nested under sections and blocks
- Configured using settings in the theme editor
- Given presets and added by merchants
- Used as [static blocks](mdc:https:/shopify.dev/docs/storefronts/themes/architecture/blocks/theme-blocks/static-blocks#statically-vs-dynamically-rendered-theme-blocks) by theme developers

Blocks render in the editor and storefront when they are referenced in [template files](mdc:.cursor/rules/templates.mdc).

### Basic block structure
```liquid
{% doc %}
  Block description and usage examples

  @example
  {% content_for 'block', type: 'block-name', id: 'unique-id' %}
{% enddoc %}

<div {{ block.shopify_attributes }} class="relative">
  <!-- Block content using block.settings -->
</div>

{% comment %}
  Note: Use Tailwind utility classes for all styling.
  Avoid custom CSS stylesheets unless absolutely necessary for complex animations or unique styling.
{% endcomment %}

{% schema %}
{
  "name": "Block Name",
  "settings": [],
  "presets": []
}
{% endschema %}
```

### Static block usage

Static blocks are theme blocks that are rendered directly in Liquid templates by developers, rather than being dynamically added through the theme editor. This allows for predetermined block placement with optional default settings.

**Basic static block syntax:**
```liquid
{% content_for 'block', type: 'text', id: 'header-announcement' %}
```

**Example: Product template with mixed static and dynamic blocks**
```liquid
<!-- templates/product.liquid -->
<div class="product-page">
  {% comment %} Static breadcrumb block {% endcomment %}
  {% content_for 'block', type: 'breadcrumb', id: 'product-breadcrumb' %}

  <div class="product-main">
    <div class="product-media">
      {% comment %} Static product gallery block {% endcomment %}
      {% content_for 'block', type: 'product-gallery', id: 'main-gallery', settings: {
        enable_zoom: true,
        thumbnails_position: "bottom"
      } %}
    </div>

    <div class="product-info">
      {% comment %} Static product info blocks {% endcomment %}
      {% content_for 'block', type: 'product-title', id: 'product-title' %}
      {% content_for 'block', type: 'product-price', id: 'product-price' %}
      {% content_for 'block', type: 'product-form', id: 'product-form' %}

      {% comment %} Dynamic blocks area for additional content {% endcomment %}
      <div class="product-extra-content">
        {% content_for 'blocks' %}
      </div>
    </div>
  </div>

  {% comment %} Static related products block {% endcomment %}
  {% content_for 'block', type: 'related-products', id: 'related-products', settings: {
    heading: "You might also like",
    limit: 4
  } %}
</div>
```

**Key points about static blocks:**
- They have a fixed `id` that makes them identifiable in the theme editor
- Settings can be overridden in the theme editor despite having defaults
- They appear in the theme editor as locked blocks that can't be removed or reordered
- Useful for consistent layout elements that should always be present
- Can be mixed with dynamic block areas using `{% content_for 'blocks' %}`

## Schema configuration

See [schemas.mdc](mdc:.cursor/rules/schemas.mdc) for rules on schemas

### Advanced schema features

#### Exclude wrapper

```json
{
  "tag": null  // No wrapper - must include {{ block.shopify_attributes }} for proper editor function
}
```

## Block implementation patterns

### Accessing block data

**Block settings:**
```liquid
{{ block.settings.text }}
{{ block.settings.heading | escape }}
{{ block.settings.image | image_url: width: 800 }}
```

**Block properties:**
```liquid
{{ block.id }}           // Unique block identifier
{{ block.type }}         // Block type name
{{ block.shopify_attributes }}  // Required for theme editor
```

**Section context:**
```liquid
{{ section.id }}         // Parent section ID
{{ section.settings.heading | escape }}
{{ section.settings.image | image_url: width: 800 }}
```

## Nested blocks implementation

### Rendering nested blocks
```liquid
<div class="relative p-4" {{ block.shopify_attributes }}>
  <h2 class="text-2xl font-bold mb-4">{{ block.settings.heading | escape }}</h2>

  <div class="flex flex-col gap-4">
    {% content_for 'blocks' %}
  </div>
</div>
```

### Nesting with layout control
```liquid
<div
  class="flex {% if block.settings.layout_direction == 'horizontal' %}flex-row justify-between items-center px-{{ block.settings.padding }}{% else %}flex-col items-{{ block.settings.alignment }} py-{{ block.settings.padding }}{% endif %}"
  {{ block.shopify_attributes }}
>
  {% content_for 'blocks' %}
</div>
```

### Presets with nested blocks
```json
{
  "presets": [
    {
      "name": "t:names.two_column_layout",
      "category": "Layout",
      "settings": {
        "layout_direction": "horizontal"
      },
      "blocks": [
        {
          "type": "text",
          "settings": {
            "text": "Column 1 content"
          }
        },
        {
          "type": "text",
          "settings": {
            "text": "Column 2 content"
          }
        }
      ]
    }
  ]
}
```

## CSS and styling

**Use Tailwind utility classes for all styling.** Avoid custom CSS stylesheets unless absolutely necessary.

### Styling with Tailwind
```liquid
<div
  class="p-4 bg-transparent rounded-lg"
  {{ block.shopify_attributes }}
>
  <h2 class="text-2xl font-bold text-foreground">
    {{ block.settings.heading | escape }}
  </h2>
</div>
```

### Dynamic styling with inline styles
```liquid
<div
  class="p-4 rounded-lg"
  style="
    padding: {{ block.settings.padding }}px;
    text-align: {{ block.settings.alignment }};
    background-color: {{ block.settings.background_color }};
  "
  {{ block.shopify_attributes }}
>
```

### Responsive design with Tailwind
```liquid
<div
  class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4 p-4 md:p-8"
  {{ block.shopify_attributes }}
>
```

## Block targeting

### Section schema for theme blocks
```json
{
  "blocks": [
    { "type": "@theme" },  // Accept all theme blocks
    { "type": "@app" }     // Accept app blocks
  ]
}
```

### Restricted block targeting
```json
{
  "blocks": [
    {
      "type": "text",
      "name": "Text Content"
    },
    {
      "type": "image",
      "name": "Image Content"
    }
  ]
}
```

## Common block patterns

### Content block
```liquid
<div class="flex flex-col gap-4 p-6 {{ block.settings.style }}" {{ block.shopify_attributes }}>
  {% if block.settings.heading != blank %}
    <h3 class="text-2xl font-bold text-foreground">{{ block.settings.heading | escape }}</h3>
  {% endif %}

  {% if block.settings.text != blank %}
    <div class="text-base text-foreground/80">{{ block.settings.text }}</div>
  {% endif %}

  {% if block.settings.button_text != blank %}
    <a href="{{ block.settings.button_url }}" class="button button-boost">
      {{ block.settings.button_text | escape }}
    </a>
  {% endif %}
</div>
```

### Media block
```liquid
<div class="relative overflow-hidden rounded-lg" {{ block.shopify_attributes }}>
  {% if block.settings.image %}
    <div class="relative w-full h-auto">
      {{ block.settings.image | image_url: width: 800 | image_tag:
         alt: block.settings.image.alt | default: block.settings.alt_text,
         class: 'w-full h-auto object-cover'
      }}
    </div>
  {% endif %}

  {% if block.settings.video %}
    <div class="relative w-full">
      {{ block.settings.video | video_tag: controls: true, class: 'w-full h-auto' }}
    </div>
  {% endif %}
</div>
```

### Layout block (container)
```liquid
<div
  class="grid gap-{{ block.settings.gap }}"
  style="grid-template-columns: repeat({{ block.settings.columns }}, minmax(0, 1fr));"
  {{ block.shopify_attributes }}
>
  {% content_for 'blocks' %}
</div>
```

## Performance best practices

### Conditional rendering
```liquid
{% liquid
  assign has_content = false
  if block.settings.heading != blank or block.settings.text != blank
    assign has_content = true
  endif
%}

{% if has_content %}
  <div class="relative p-4" {{ block.shopify_attributes }}>
    <!-- Content here -->
  </div>
{% endif %}
```

## Examples referenced

[text.liquid](mdc:.cursor/rules/examples/block-example-text.liquid) - Basic content block from existing project
[group.liquid](mdc:.cursor/rules/examples/block-example-group.liquid) - Container with nested blocks from existing project

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lukaivanovic) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
