## astro-strapi-blocks

> Astro + Strapi Blocks — StrapiBlocks, data, theme (extend/overwrite), custom blocks


# Astro Strapi Blocks

- **Import:** `StrapiBlocks` from `@sensinum/astro-strapi-blocks`. **`data`:** raw Strapi 5 Blocks array (`StrapiBlockField`); use the same field the API already returns (e.g. `page.content`, `item.description`).

- **Theme:** `theme={{ extend: { … } }}` appends/merges with defaults; `overwrite` replaces class arrays at each path. Library order: `overwrite` first, then `extend`. For nested list markers, set `list.indent.ordered` / `unordered` as per-level class arrays; see package README.

- **Custom components:** `blocks={{ heading: MyHeading, … }}`. Each Astro override must match the block’s props (`data`, `theme`, and type-specific props in README). Use `renderPropertyClasses` / `getPropertyClass` from the package when reusing theme paths.

- **New CMS block types** need Strapi schema support; this component only dispatches built-in `type` values—combine with dynamic zones and parent switches for non-standard blocks.

- Deeper context: repository `.ai/astro-strapi-blocks/SKILL.md` and root `README.md`.

---
> Source: [VirtusLab-Open-Source/astro-strapi-blocks](https://github.com/VirtusLab-Open-Source/astro-strapi-blocks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
