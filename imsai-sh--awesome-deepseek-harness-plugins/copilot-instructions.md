## awesome-deepseek-harness-plugins

> - The plugin catalog uses the canonical public path `/plugin`.

# Repository instructions

## Public URL stability

- The plugin catalog uses the canonical public path `/plugin`.
- Plugin detail pages use the canonical public path `/plugin/:owner/:name` (singular `plugin`).
- Plugin API routes use `/api/plugin` and `/api/plugin/:owner/:name`.
- Treat public route paths as permanent SEO contracts. Do not rename or remove them without explicit user approval and a migration plan covering permanent redirects, canonical URLs, and existing inbound links.
- When replacing an already-published route, keep a permanent redirect from the old path to the canonical path.

## Responsive web support

- The website supports both desktop and mobile devices. Treat both layouts as first-class release requirements.
- Start from the narrow layout and progressively enhance it. Do not rely on fixed desktop widths, hover-only interactions, or desktop-only information hierarchy.
- For every UI, layout, spacing, or typography change, design and verify both desktop and touch-enabled mobile viewports; do not approve a change based on desktop appearance alone.
- As a minimum, run `npm run test:visual` and check representative viewports around 1440×900, 390×844, and 320×568. Confirm there is no unintended page-level horizontal overflow, clipping, overlap, or content hidden behind sticky UI or safe areas.
- Primary buttons, icon buttons, tabs, filters, and other repeated controls must provide at least a 44×44 CSS-pixel touch target on mobile. Inputs must use a 16px or larger font on mobile so iOS does not zoom the page on focus.
- Keep body and explanatory copy readable on mobile (normally at least 12px for compact metadata and 14px for prose). Prefer reflowing or intentionally scrollable local regions over shrinking text to make desktop layouts fit.
- Horizontal chip, tab, table, code, and README overflow must stay inside an intentional local scroller with touch panning; the document itself must never scroll horizontally.
- Preserve task priority when content stacks: primary actions and safety information come before secondary metadata, and long-form content comes afterward.
- When changing responsive behavior, extend `apps/web/scripts/visual-check.mjs` with a regression assertion for the affected mobile interaction or layout invariant.

---
> Source: [imsai-sh/awesome-deepseek-harness-plugins](https://github.com/imsai-sh/awesome-deepseek-harness-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-15 -->
