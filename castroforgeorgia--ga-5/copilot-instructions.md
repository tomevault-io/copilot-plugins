## ga-5

> - Jekyll drives page rendering; collections and layouts live under [`content/`](mdc:content) and [`_layouts/`](mdc:_layouts).

# Castro for Georgia Frontend Overview
- Jekyll drives page rendering; collections and layouts live under [`content/`](mdc:content) and [`_layouts/`](mdc:_layouts).
- Vite builds front-end assets from [`assets/`](mdc:assets); run `npm run dev` (Vite + Jekyll) or `npm run build` for production bundles before deployment.
- Tailwind and custom styles compile from [`_sass/input.css`](mdc:_sass/input.css) via PostCSS; avoid editing generated CSS in [`assets/css/`](mdc:assets/css).
- Content configuration stays in [`_data/settings.yml`](mdc:_data/settings.yml) and [`_config.yml`](mdc:_config.yml); adjust localization or site metadata there.
- The `_site/` directory is build output—do not edit files inside it.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CastroForGeorgia) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
