## medal-forge

> When adding or renaming a material preset:

# Project Notes

## Material Preset Workflow

When adding or renaming a material preset:

- Add or update the preset in `lib/materials.ts`.
- Every preset must include a `previewSrc` that points to a committed preview asset.
- Store built-in material preview assets in `public/material-previews/`.
- Preview assets should render a small sphere-like material swatch that visually reflects the material color, roughness, metalness, and finish.
- Keep preview filenames stable and kebab-case, for example `brushed-gold.svg`.
- If a future feature generates new materials, generate and save the matching preview asset at creation time, then persist the material record with its preview path.
- Do not leave a material preset without a preview image; the custom material dropdown depends on it.

---
> Source: [CatsJuice/medal-forge](https://github.com/CatsJuice/medal-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
