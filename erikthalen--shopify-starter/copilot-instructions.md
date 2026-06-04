## shopify-starter

> Every image rendered in Liquid **must** use the `image` snippet:

# Project Guidelines

## Images

Every image rendered in Liquid **must** use the `image` snippet:

```liquid
{% render 'image', image: product.featured_image %}
```

Never use `image_tag`, `image_url`, or raw `<img>` tags directly. The snippet handles responsive srcsets, WebP conversion, focal point cropping, aspect ratios, and lazy loading correctly.

---
> Source: [erikthalen/shopify-starter](https://github.com/erikthalen/shopify-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
