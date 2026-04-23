## greentech-storyblok-astro

> The project uses Storyblok as a headless CMS. Storyblok components are mapped to Astro components in the [astro.config.mjs](mdc:astro.config.mjs) file.

# Storyblok Components

The project uses Storyblok as a headless CMS. Storyblok components are mapped to Astro components in the [astro.config.mjs](mdc:astro.config.mjs) file.

## Component Mapping

Components are mapped in the `storyblok` integration configuration:

```javascript
components: {
  button: 'storyblok/Button',
  page: 'storyblok/Page',
  hero: 'storyblok/Hero',
  grid: 'storyblok/Grid',
  row: 'storyblok/Row',
  text: 'storyblok/Text',
  richText: 'storyblok/RichText',
  carousel: 'storyblok/Carousel',
  card: 'storyblok/Card/index',
  list: 'storyblok/List',
  navItem: 'storyblok/NavItem',
  component: 'storyblok/Component/index',
  link: 'storyblok/Link',
  badge: 'storyblok/Badge',
  form: 'storyblok/Form/index',
  input: 'storyblok/Input',
  accordion: 'storyblok/Accordion',
  visual: 'storyblok/Visual',
  tabs: 'storyblok/Tabs',
  tabItem: 'storyblok/TabItem',
  gallery: 'storyblok/Gallery',
}
```

## Component Definitions

All Storyblok components are located in the [src/storyblok/](mdc:src/storyblok) directory.

## Type Generation

TypeScript types for Storyblok components are generated using:
```bash
npm run storyblok:generate-types
```

This pulls component definitions from Storyblok and generates TypeScript types in [src/types/component-types-sb.d.ts](mdc:src/types/component-types-sb.d.ts).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cozmic-creatives) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
