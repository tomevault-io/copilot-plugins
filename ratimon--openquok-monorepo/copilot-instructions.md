## company-brand-icons

> Keep branded icons in sync for public company, competitor, agent, and MCP entries


# Brand icon sync

When adding a new public company or product entry under competitors, agents, or MCP constants:

- Add or reuse a branded icon in `web/src/data/icons/branded-icons.ts` first.
- Reference icons through the registry with `icons.SomeBrand.name` rather than hardcoded strings.
- Keep icon names aligned with the content constant so new entries render consistently in hubs, compare pages, and detail pages.
- If the new entry appears in a public selector, card, or compare surface, make sure the view model carries the icon through instead of re-deriving it in page markup.
- Prefer inline SVG paths in the icon registry for vector logos; use local image assets only when vector artwork is not practical.

---
> Source: [Ratimon/openquok-monorepo](https://github.com/Ratimon/openquok-monorepo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
